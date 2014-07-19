---
layout: post
title: "MVC with GWT: creating and firing custom events with GWT"
date: 2010-06-26 17:30:00
---

GWT is obviously able to handle events, for instance `ClickEvent` fired by `Button`, or `ResizeEvent` fired by `Window`. Wouldn't it be nice to be able to create custom events? What I have in mind is to create Models that can be binded to Views. I won't go that far in this post, but instead will just try to figure out:

* how GWT2 is handling events,
* how we can create custom events and use them.

We will start with a very simple Model, and fire an event whenever its property, called field, is modified.

Here is our simple model:

{% highlight java %}
public class SimpleModel
{
    private String field;

    public SimpleModel(String field)
    {
        this.field = field;
    }

    public String getField()
    {
        return field;
    }

    public void setField(String field)
    {
        this.field = field;
    }
}
{% endhighlight %}


How is GWT handling events?
---------------------------

If we look all the way up to the `Widget` class, we notice that GWT is able to handle two kinds of events.

* DOM events, that are native events fired by the underlying html elements:

{% highlight java %}
/**
 * Adds a native event handler to the widget and sinks the corresponding
 * native event. If you do not want to sink the native event, use the
 * generic addHandler method instead.
 *
 * @param <H> the type of handler to add
 * @param type the event key
 * @param handler the handler
 * @return {@link HandlerRegistration} used to remove the handler
 */
protected final <H extends EventHandler> HandlerRegistration
        addDomHandler(final H handler, DomEvent.Type<H> type) {
    assert handler != null : "handler must not be null";
    assert type != null : "type must not be null";
    sinkEvents(Event.getTypeInt(type.getName()));
    return ensureHandlers().addHandler(type, handler);
}
{% endhighlight %}


* "other" events:

{% highlight java %}
/**
 * Adds this handler to the widget.
 *
 * @param <H> the type of handler to add
 * @param type the event type
 * @param handler the handler
 * @return {@link HandlerRegistration} used to remove the handler
 */
protected final <H extends EventHandler> HandlerRegistration addHandler(
        final H handler, GwtEvent.Type<H> type) {
    return ensureHandlers().addHandler(type, handler);
}
{% endhighlight %}


Custom events are what we are looking for! The problem is that, well, our simple model does not by any stretch of the imagination qualify as a `Widget`. So we cannot have our model inherit from the `Widget` class.

If we look a bit more at the two methods above, we notice they are both calling a method called `ensureHandlers()`.

{% highlight java %}
/**
 * Ensures the existence of the handler manager.
 *
 * @return the handler manager
 * */
HandlerManager ensureHandlers() {
    return handlerManager == null ?
        handlerManager = new HandlerManager(this) : handlerManager;
}
{% endhighlight %}


This method is returning a `HandlerManager`. If you go look in the code or the documentation, a `HandlerManager` is "responsible for adding handlers to event sources and firing those handlers on passed in events.". That's precisely what we were looking for. So all we have to do is to add a `HandlerManager` to our `SimpleModel`.


Creating a `BaseModel` class that is able to fire events
--------------------------------------------------------

In order to handle events in `BaseModel`, we basically need two methods :

* a method to add handlers to the handler manager,
* a method to fire events.

{% highlight java %}
public class BaseModel
{
    private HandlerManager handlerManager = new HandlerManager(this);

    public <H extends EventHandler> HandlerRegistration
        addHandler(GwtEvent.Type<H> type, final H handler)
    {
        return handlerManager.addHandler(type, handler);
    }

    public void fireEvent(GwtEvent<?> event)
    {
        handlerManager.fireEvent(event);
    }
}
{% endhighlight %}

Perfect. Now all we need to do is to extend the `SimpleModel` so that it fires events.


Adding an event handler and firing events from `SimpleModel`
------------------------------------------------------------

At first we need to define what the `EventHandler` will be. We want to know when the value of field is changed, so it should be something like :

{% highlight java %}
public interface FieldChangedHandler extends EventHandler
{
    public void onFieldChanged(String newValue, String oldValue);
}
{% endhighlight %}

Here is the final `SimpleModel` class, firing a `GwtEvent` whenever any of its field is changed:

{% highlight java %}
public class SimpleModel extends BaseModel
{
    private String field;

    public String getField()
    {
        return field;
    }

    public void setField(String field)
    {
        String oldValue = this.field;
        String newValue = field;
        this.field = field;
        this.fireEvent(new FieldChangedEvent(oldValue, newValue));
    }

    public HandlerRegistration addFieldChangedHandler(FieldChangedHandler handler)
    {
        return addHandler(handler, FieldChangedEvent.getType());
    }

    public interface FieldChangedHandler extends EventHandler
    {
        public void onFieldChanged(String newValue, String oldValue);
    }

    public static class FieldChangedEvent extends GwtEvent<FieldChangedHandler>
    {
        private final String oldValue, newValue;

        public FieldChangedEvent(String newValue, String oldValue)
        {
            this.newValue = newValue;
            this.oldValue = oldValue;
        }

        /**
         * The event type.
         */
        private static Type<FieldChangedHandler> TYPE = new Type<FieldChangedHandler>();

        /**
         * Handler hook.
         *
         * @return the handler hook
         */
        public static Type<FieldChangedHandler> getType()
        {
            if (TYPE == null)
                TYPE = new Type<FieldChangedHandler>();
            return TYPE;
        }

        @Override
        protected void dispatch(FieldChangedHandler handler)
        {
            handler.onFieldChanged(newValue, oldValue);
        }

        @Override
        public com.google.gwt.event.shared.GwtEvent.Type<FieldChangedHandler> getAssociatedType()
        {
            return TYPE;
        }
    }
}
{% endhighlight %}


In order to observe this simple model, just add something like this where needed in your code:

{% highlight java %}
mySimpleModel.addFieldChangedHandler(new FieldChangedHandler()
{
    @Override
    public void onFieldChanged(String newValue, String oldValue)
    {
        // TODO Something
    }
});
{% endhighlight %}

That's great, now we have a SimpleModel firing events whenever its unique property is changed. But as you can imagine, defining an `EventHandler` and a `GwtEvent` for each and every field of a model can quickly become tedious work...
