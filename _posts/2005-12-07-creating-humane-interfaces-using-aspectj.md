---
layout: post
title: Creating humane interfaces using AspectJ
date: 2005-12-07 13:39:33.000000000 -08:00
categories:
- Technology
tags: []
status: publish
type: post
published: true
color: navy
---
Martin Fowler blogged about "[minimal](http://martinfowler.com/bliki/MinimalInterface.html)" vs. "[humane](http://martinfowler.com/bliki/HumaneInterface.html)" interface. A minimal interface provides only the basic methods enabling, but not providing, clients to write convenience methods. A humane interface, on the other hand, considers typical uses of the interface and provides convenience methods as a part of the interface itself. Providing convenience methods through a wrapper (similar to the `java.util.Collections` class) isn’t as natural as providing them as a part of the interface itself. Further, using a wrapper for accessing minor convenience methods (such as `List.first()` and `List.last()`, see below) is difficult to justify given the extra code you need to write.

Martin asks "what's the basis for deciding what should be added to a humane interface?". This is a really important question. I have seen fat "humane" interfaces, where many methods had dubious value and only increased the complexity of those interfaces. I believe creating good humane interfaces requires that

*   Use cases form the basis for deciding what should be added to a humane interface. This means each project may need different humane interfaces for the same underlying concept.
*   Convenience methods for each use case (or coherent set of use cases) be contributed by separate humanizing modules. This allows keeping the core interfaces clean, while adapting them to project-specific needs by including appropriate modules.

Languages such as Java make the separation hard to achieve, since a type needs to define everything in one place and interfaces cannot contain method implementations. We can overcome these limitations using [AspectJ](http://eclipse.org/aspectj). Here core interfaces follow the minimalist approach and aspects introduce convenience methods using inter-type declarations.

For example, you can humanize the access to the `List` interface using an aspect such as follows:

{% highlight java %}
public aspect HumanizeListAccess {
    public E List<E>.first() {
        return size() > 0 ? get(0) : null;
    }

    public E List<E>.last() {
        return size() > 0 ? get(size()-1) : null;
    }
}
{% endhighlight %}

The aspect introduces two new methods, along with their implementations, to the `List` interface. Now you can use the interface as follows:

{% highlight java %}
List<string> tickers = new ArrayList</string><string>();

tickers.add("GOOG");
tickers.add("SUNW");
tickers.add("YHOO");

firstTicker = tickers.first();

latestTicker = tickers.last();
{% endhighlight %}

The only problem with the above implementation: it works only if you weave this aspect into rt.jar!

Here is another example of humanizing an interface the AOP way (and does not require weaving into rt.jar). Consider the following `Inventory` interface:

{% highlight java %}
public interface Inventory {

    public void addItem(Item item, int count);

    public void removeItem(Item item, int count);

    public Collection<item> getItems();

    public int getItemCount(Item item);
}
{% endhighlight %}

You can humanize this interface to provide price awareness using an aspect as follows:

{% highlight java %}
public aspect HumanizeInventoryPriceAwareness {

    public Item Inventory.getLeastExpensiveItem() {
        float leastPrice = Float.MAX_VALUE;
        Item leastExpensiveItem = null;

        for(Item item : getItems()) {
            if(item.getPrice() < leastPrice) {
                leastPrice = item.getPrice();
                leastExpensiveItem = item;
            }
        }

        return leastExpensiveItem;
    }

    public Item Inventory.getMostExpensiveItem() {
        ... similar to getLeastExpensiveItem()
    }

    public Collection<Item> Inventory.inRangeItems(float lowPrice, float highPrice) {
        Collection</item><item> inRangeItems = new ArrayList</item><item>();

        for(Item item : getItems()) {
            if((item.getPrice() >= lowPrice)
               || (item.getPrice() < = highPrice)) {
                inRangeItems.add(item);
            }
        }

        return inRangeItems;
    }

    public Collection<Item> Inventory.cheaperThanItems(float price) {
        return inRangeItems(0, price);
    }

    public Collection</item><item> Inventory.expensiveThanItems(float price) {
        return inRangeItems(price, Float.MAX_VALUE);
    }
}
{% endhighlight %}

The `HumanizeInventoryPriceAwareness` aspect uses inter-type declarations introducing new methods, along with their implementations, to the Inventory interfaces. Notice, how `cheaperThanItems()` and `expensiveThanItems()` themselves use the convenience methods introduced by the same aspect.

Now you can use the humanized Inventory interface as follows:

{% highlight java %}
Inventory inventory = ...

Item leastExpensive = inventory.getLeastExpensiveItem();

Collection</item><item> itemsForFriends = inventory.inRangeItems(20f, 50f);

Collection</item><item> itemsForSpouse = inventory.expensiveThanItems(400f);
{% endhighlight %}

Note that nothing prevents you from including the aspects in the same source file as the interface or even as nested aspects inside the interface. If you do so, you will get compartmentalization instead of full separation.

So far, we relied on static crosscutting alone. We can implement more complex convenience methods when we throw in dynamic crosscutting. Here dynamic crosscutting tracks the required information and static crosscutting exposes it. For example, here we track access to the inventory and expose the last accessed item through a method:

{% highlight java %}

public aspect HumanizeInventoryTracking {

    private Item Inventory.lastAccessedItem;

    public Item Inventory.getLastAccessedItem() {
        return lastAccessedItem;
    }

    after(Inventory inventory, Item item) returning
        : (execution(* Inventory.addItem(Item, ..))
           || execution(* Inventory.removeItem(Item,..)))
           && this(inventory)
           && args(item, ..) {
        inventory.lastAccessedItem = item;
    }
}
{% endhighlight %}

You can follow this pattern to implement more interesting methods such as `addedSinceItems(Date)`, `mostAccessedItem()`, and `leastAccessedItem()`.

Implementing humane interfaces with the help of AOP allows developing and maintaining each module separately. It also allows easy customization of an interface to meet a project’s needs by simply choosing a suitable set of aspects. For example, if price awareness isn’t required in your project simply exclude the `HumanizeInventoryPriceAwareness` aspect.
