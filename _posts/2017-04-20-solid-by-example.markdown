---
layout: post
title: SOLID by example
---

When learning about the SOLID principles I haven't come across a single source that applies all the principles at once in a single example. So in this article that's exactly what I aim to achieve. I assume that you know what the principles are and you may even try to apply them on a regular basis. Unfortunately without a concrete example of all of the principles working together it can be difficult to know what your code should look like, and what the benefits are.

Because this is an example the results might feel somewhat "overkill" but when applied to larger and more complex real world domains you begin to reap benefits in terms of flexibility, testability and readability.

The example scenario that we'll be developing is thus:

Clients want to be able to send a payload with a list of VIN codes (vehicle identification numbers) as JSON or XML and expect a response containing the VIN with the price and the year the vehicle was manufactured, in the format they sent the payload in. Two example requests and responses would be:

#### JSON

Request

{% highlight json %}
[
    {"vin": "1FADP3F25GL393953"},
    {"vin": "1FADP3F28GL401740"}
]
{% endhighlight %}

Response

{% highlight json %}
[
    {
        "vin": "1FADP3F25GL393953",
        "price": "12345.65",
        "manufactured_date": "2015-04-12"
    },
    {
        "vin": "1FADP3F28GL401740",
        "price": "4566.89",
        "manufactured_date": "2012-08-01"
    }
]
{% endhighlight %}

#### XML

Request

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<vehicles>
    <vehicle><vin>1FADP3F25GL393953</vin></vehicle>
    <vehicle><vin>1FADP3F28GL401740</vin></vehicle>
</vehicles>
{% endhighlight %}

Response

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<vehicles>
    <vehicle>
        <vin>1FADP3F25GL393953</vin>
        <price>12345.65</price>
        <manufactured-date>2015-04-12</manufactured-date>
    </vehicle>
    <vehicle>
        <vin>1FADP3F28GL401740</vin>
        <price>4566.89</price>
        <manufactured-date>2012-08-01</manufactured-date>
    </vehicle>
</vehicles>
{% endhighlight %}

For the sake of brevity imagine that the data is well structured, validated, and the format is strictly as it looks there at all times. The task we're assigned is purely to parse the request, and generate the response.

As well as following along with the article, you can find the complete code structured as a project [on github](https://github.com/mhotchen/SOLID-by-example).

The first thing to tackle would be the vehicle data structure at the heart of the whole thing. Looking at the request and response we can see that there are three fields, the VIN, price, and manufactured date. Initially it might be tempting to have the price and manufactured date as optional values because they're unused on the request as in the following example:

{% highlight php %}
<?php

class Vehicle
{
    private $vin;
    private $price;
    private $manufacturedDate;

    public function __construct(
        string $vin,
        string $price = null,
        DateTime $manufacturedDate = null
    ) {
        $this->vin = $vin;
        $this->price = $price;
        $this->manufacturedDate = $manufacturedDate;
    }
}
{% endhighlight %}

But this violates the S in SOLID (single responsibility principle). We've given this object two responsibilities: the first is to hold a request which simply contains a VIN, and the second is to hold a vehicle. We're better off separating the two as they can easily diverge.

{% highlight php %}
<?php

class VinLookupRequest
{
    private $vin;

    public function __construct(string $vin)
    {
        $this->vin = $vin;
    }
}

class Vehicle
{
    private $vin;
    private $price;
    private $manufacturedDate;

    public function __construct(
        string $vin,
        string $price,
        DateTime $manufacturedDate
    ) {
        $this->vin = $vin;
        $this->price = $price;
        $this->manufacturedDate = $manufacturedDate;
    }
}
{% endhighlight %}

Now the request can diverge quite easily, for example by giving a license plate instead. The request could probably do with an interface to allow requests of various different fields to be handled, but that's outside the scope of the task at hand and easy to add later if necessary, so let's ignore that for now (YAGNI).

Next we can look at the formats for the request/response. We know that both work with XML and JSON. The reading will convert JSON or XML to an array of VinLookupRequest objects, whilst writing will convert an array of Vehicle objects in to JSON or XML. It feels natural to approach this by having an interface that both the JSON and XML versions can implement, so let's design that now:

{% highlight php %}
<?php

interface VinLookupRequestReader
{
    /**
     * @param string $payload
     * @return VinLookupRequest[]
     */
    public function read(string $payload): array;
}

interface VehicleWriter
{
    /**
     * @param Vehicle[] $vehicles
     * @return string
     */
    public function write(array $vehicles): string;
}
{% endhighlight %}

It might have been tempting to have one interface with both the read/write methods instead of two separate ones (for example a Converter which converts to/from the request/response format and the domain format) but this violates the single responsibility principle and makes it difficult to modify one behaviour without affecting the other. It also means that, for example, if you want to implement just a writer that spits out HTML for debugging, then you'll have to mock or stub the read method which would violate the Liskov substitution principle since the read would no longer be behaving in line with its contract.

Now that we've designed the contracts for reading and writing, let's implement them:

{% highlight php %}
<?php

final class JsonVinLookupRequestReader implements VinLookupRequestReader
{
    public function read(string $payload): array
    {
        return array_map(
            function ($vehicleLookupRequest) {
                return new VinLookupRequest(
                    $vehicleLookupRequest->vin
                );
            },
            json_decode($payload)
        );
    }
}

final class XmlVinLookupRequestReader implements VinLookupRequestReader
{
    private $xmlReader;
    
    public function __construct(XMLReader $xmlReader)
    {   
        $this->xmlReader = $xmlReader;
    }
    
    public function read(string $payload): array
    {   
        $response = [];
        $this->xmlReader->xml($payload); 
        while ($this->xmlReader->read()) { 
            if ($this->xmlReader->nodeType == XMLReader::END_ELEMENT) {
                continue;
            }
        
            if ($this->xmlReader->name === 'vin') {
                $response[] = new VinLookupRequest(
                    $this->xmlReader->readString()
                );
            }
        }
        
        return $response;
    }
}

final class JsonVehicleWriter implements VehicleWriter
{
    public function write(array $vehicles): string
    {
        return json_encode(array_map(
            function (Vehicle $vehicle) {
                return [
                    'vin' => $vehicle->getVin(),
                    'price' => $vehicle->getPrice(),
                    'manufactured_date' => $vehicle->getManufacturedDate()
                ];  
            },
            $vehicles
        )); 
    }
}

final class XmlVehicleWriter implements VehicleWriter
{
    private $xmlWriter;

    public function __construct(XMLWriter $xmlWriter)
    {   
        $this->xmlWriter = $xmlWriter;
    }   

    public function write(array $vehicles): string
    {
        $this->xmlWriter->startDocument('1.0', 'UTF-8');
        $this->xmlWriter->startElement('vehicles');
        foreach ($vehicles as $vehicle) {
            $this->xmlWriter->startElement('vehicle');
            $this->xmlWriter->writeElement('vin', $vehicle->getVin());
            $this->xmlWriter->writeElement('price', $vehicle->getPrice());
            $this->xmlWriter->writeElement(
                'manufactured-date',
                $vehicle->getManufacturedDate()
            );  
            $this->xmlWriter->endElement();
        }   
        $this->xmlWriter->endElement();
        $this->xmlWriter->endDocument();

        return $this->xmlWriter->outputMemory(true);
    }
}
{% endhighlight %}

Nice! That looks SOLID to me. We have segregated interfaces with a single responsibility. They're open for extension by way of adding new readers/writers to the system, without having to modify them.

Now I want to talk about the XMLReader/XMLWriter injection. Why are these injected? If you thought it's for dependency inversion then you're wrong. These dependencies _aren't_ inverted. An inverted dependency is one where you're dependent on an abstraction of a behavior (through an interface or abstract class usually), not the behavior itself. These classes remain wholly coupled with the XMLReader and XMLWriter and _that's okay_! These classes themselves are the abstractions that implement an interface that the rest of the system depends on.

So why are the XMLReader/XMLWriter classes injected if not for dependency inversion? The answer is twofold: first if they weren't injected then these classes violate the single responsibility principle. They would have to construct the XMLReader/XMLWriter _and_ use them. Second is a principle closely related to SOLID called Inversion of Control (IoC). By inverting control of object creation we're free to alter the object graph at runtime. Why is that important? Well in this case it would allow us to mock an XMLWriter in the test suite and ensure it calls startElement with 'vehicle' as a parameter exactly three times if three vehicles were passed in.

## Which reader/writer to use?

Now that we have our nicely separated readers and writers we need to decide which strategy our system will use for reading/writing data.

Let's start out by building our type detection:

{% highlight php %}
<?php

interface IsType
{
    public function isType(string $payload): bool;
}

final class IsJson implements IsType
{
    public function isType(string $payload): bool
    {
        return in_array($payload[0], ['{', '[']);
    }
}

final class IsXml implements IsType
{
    public function isType(string $payload): bool
    {
        return $payload[0] === '<';
    }
}

{% endhighlight %}

This is pretty straight forward. In the name of brevity I've used an insanely naive assumption: if the request payload starts with a less than symbol (<) then it's XML and if it starts with an opening square bracket ([) or curly brace ({) it's JSON.

Now we need a way of connecting the payload type detection with which reader/writer we want to use:

{% highlight php %}
<?php

interface ReaderStrategy
{
    public function canUseStrategy(): bool;
    public function getReader(): VinLookupRequestReader;
}

interface WriterStrategy
{
    public function canUseStrategy(): bool;
    public function getWriter(): VehicleWriter;
}

final class Strategy implements ReaderStrategy, WriterStrategy
{
    private $payloadType;
    private $payload;
    private $reader;
    private $writer;

    public function __construct(
        IsType $payloadType,
        string $payload,
        VinLookupRequestReader $reader,
        VehicleWriter $writer
    ) {
        $this->payloadType = $payloadType;
        $this->payload = $payload;
        $this->reader = $reader;
        $this->writer = $writer;
    }

    public function canUseStrategy(): bool
    {
        return $this->payloadType->isType($this->payload);
    }

    public function getReader(): VinLookupRequestReader
    {
        return $this->reader;
    }

    public function getWriter(): VehicleWriter
    {
        return $this->writer;
    }
}
{% endhighlight %}

It's entirely possible that in the future the writer will be based off something else entirely, such as an Accept header in the request. That's why the payload is in the object construction rather than in the canUseStrategy method's parameters, it allows the writer and reader to eventually be broken in to two different strategy objects if they must be without affecting how the object is used. This is one of the beauties of segregating your interfaces.

Finally we need a way of putting it all together. Following an IoC approach I'll build an ugly object graph building system:

{% highlight php %}
<?php

final class IocContainer
{
    public function getJsonVinLookupRequestReader(): JsonVinLookupRequestReader
    {
        return new JsonVinLookupRequestReader();
    }

    public function getXmlVinLookupRequestReader(): XmlVinLookupRequestReader
    {
        return new XmlVinLookupRequestReader(new XMLReader());
    }

    public function getJsonVehicleWriter(): JsonVehicleWriter
    {
        return new JsonVehicleWriter();
    }

    public function getXmlVehicleWriter(): XmlVehicleWriter
    {
        return new XmlVehicleWriter(new XMLWriter());
    }

    public function getVinLookupRequestReaderStrategy(
        string $payload
    ): ReaderStrategy {
        return $this->getStrategy($payload);
    }

    public function getVehicleWriterStrategy(string $payload): WriterStrategy
    {
        return $this->getStrategy($payload);
    }

    private function getStrategy(string $payload): Strategy
    {
        foreach ($this->getStrategies($payload) as $strategy) {
            if ($strategy->canUseStrategy()) {
                return $strategy;
            }
        }

        // Error handling/validation would be done based on this
        throw new Exception("Request payload is in unknown format");
    }

    private function getStrategies(string $payload): array
    {
        return [
            new Strategy(
                new IsJson(),
                $payload,
                $this->getJsonVinLookupRequestReader(),
                $this->getJsonVehicleWriter()
            ),
            new Strategy(
                new IsXml(),
                $payload,
                $this->getXmlVinLookupRequestReader(),
                $this->getXmlVehicleWriter()
            ),
        ];
    }
}
{% endhighlight %}

I want to stress the importance of using a proper IoC system like Pimple, PHP-DI, Aura.Di, etc. because these allow for a clear separation of concerns with features like modules and autowiring.

By using an IoC container it means the object graph can vary depending on, well, anything. The most useful case is for unit testing versus running the application. We can for example add stub strategies to our tests that always return a desired result, create an IsType class that always returns false to test failure scenarios, and so forth.

## Using it

Finally, we need to use it. I'm going to use some non-existent classes in the following code; some pseudo request/response objects that would be found in a typical framework and a repository for looking up the actual vehicles in. Here is what the controller might look like:

{% highlight php %}
<?php

class VinLookupRequestController
{
    private $repository;
    private $reader;
    private $writer;

    public function __construct(
        VehicleRepository $repository,
        VinLookupRequestReader $reader,
        VehicleWriter $writer
    ) {
        $this->repository = $repository;
        $this->reader = $reader;
        $this->writer = $writer;
    }

    public function handle(Request $request, Response $response)
    {
        $vins = $this->reader->read($request->getBody());
        $vehicles = $this->repository->findByVins($vins);
        $response->setBody($this->writer->write($vehicles));
    }
}
{% endhighlight %}

This is pretty straight forward, it doesn't care about which reader/writer to use so the controller can focus on simply looking the data up in the DB. Of course this is very simplistic compared to a real world domain but the article would have diminishing returns if I was to discuss an entire application.

The final piece would be having the controller built inside the IoC container which your framework/library of choice would then retrieve based upon the request:

{% highlight php %}
<?php

class IocContainer
{
    /** ... SNIP ... **/

    public function getController(Request $request): VinLookupRequestController
    {
        $payload = $request->getBody();

        // Skipping doing any URL parsing etc. since there's only one controller

        return new VinLookupRequestController(
            $this->getRepository(),
            $this
                ->getVinLookupRequestReaderStrategy($payload)
                ->getReader(),
            $this
                ->getVehicleWriterStrategy($payload)
                ->getWriter()
        );
    }
}
{% endhighlight %}

And that's it. Now, even the controller can be tested easily because it's only depending on some easy to work with interfaces.

## So how did we benefit?

Moving back to the benefits from a SOLID perspective. Taken in isolation each of the SOLID principles is useful, but the real benefit comes when you see all of them working together. Let's go through each principle against what we've designed and see how it all fits together:

### Single responsibility principle

Ignoring the object graph creation which would be better handled by a real library, the rest follows this principle nicely. Each object does only one thing and can be easily tested as an individual part. Because each object has only one responsibility it makes the next principle, Open/closed principle, much easier to adhere to.

### Open/closed principle

Firstly the IsType interface and implementations mean we can add additional types without ever having to modify an existing class. Compare that, with, say an object that has isXml, isJson, etc. as methods (violating the single responsibility principle). When it comes time to create a test version that always returns a certain value, it becomes much harder and more brittle if you're extending that class and modifying each method. Moving in to the strategies you can see that new strategies can be added easily since it delegates all of the work to other objects, another way of keeping your system extensible without needing modification.

### Liskov substitution principle

By having separate interfaces for the WriterStrategy and ReaderStrategy it means that if these need to deviate in how they decide on which strategy to use then you won't need to create&mdash;for example&mdash;a strategy that works on the Accept header for the writer that won't implement the getReader method. Here's an example of what I mean:

{% highlight php %}
<?php

interface Strategy
{
    public function canUseStrategy(): bool;
    public function getReader(): VinLookupRequestReader;
    public function getWriter(): VehicleWriter;
}

class AcceptHeaderJsonStrategy implements Strategy
{
    private $writer;

    public function __construct(VehicleWriter $writer)
    {
        $this->writer = $writer;
    }

    public function canUseStrategy(): bool
    {
        return getallheaders()['Accept'] === 'application/json';
    }

    public function getReader(): VinLookupRequestReader
    {
        throw new Exception("Not implemented");
    }

    public function getWriter(): VehicleWriter
    {
        return $this->writer;
    }
}
{% endhighlight %}

Here, because there wasn't a proper separation of the interface you end up having to violate the Liskov substitution principle in the getReader method, because that can't be implemented against the request's Accept header.

### Interface segregation principle

As just mentioned by separating the ReaderStrategy and WriterStrategy we can avoid violating the Liskov substitution principle. In addition it allows us to grow the reader and writer strategies in different directions should that need arise, or to implement just the part we're interested in when it comes to writing tests. In the IoC container we have a method getVinLookupRequestReaderStrategy and getVehicleWriterStrategy which both technically return the same object, but because we're only using one of the implemented interfaces at a time we've given ourselves freedom of choice when it comes to whether these are always the same or not.

### Dependency inversion principle

The final one really shines when it comes to the Strategy class. By keeping all dependencies as contracts rather than expecting concrete implementations it means that the Strategy class can go largely untouched as we add more strategies, or improve our data detection (going hand-in-hand with the open/closed principle). We're also easily able to build fake dependencies because it's not worried about having a real, concrete object that you then have to extend and modify, or mock. We only have a small amount of behaviour expected within each contract/interface thanks to our segregated interfaces and each object having only a single responsibility.

## Closing thoughts

The SOLID principles do have some flaws:

* The overall discoverability of an application is harder thanks to the segregation on contracts rather than seeing what concrete classes are being used
* It has a higher upfront cost of development for small and medium projects
* It's more verbose in general

Overall however, when all of them are used in unison instead of individually there are some very big wins:

* It becomes trivial to test all of the code from the lowest level to the highest
* The code is less brittle because the inheritance graph becomes flatter (typically an interface and its concrete implementations, rather than multiple levels of inheritance), and you won't see methods being overridden, only interfaces and abstract ones being defined
* The code is more malleable and welcoming of change
* The project will stay healthy for much longer

As a set of principles the sum is larger than the parts. Each principle is very useful by itself but when they're brought together it allows you to create a software project that can live, breathe, grow and change.
