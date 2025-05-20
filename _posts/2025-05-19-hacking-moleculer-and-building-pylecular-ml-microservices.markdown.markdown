---
layout: post
title:  "Hacking Moleculer and Building Pylecular: A Journey into ML/AI Microservices"
date:   2025-05-19 01:00:00 -0300
categories: [microservices, machine-learning]
tags: [python, moleculer, pylecular, ml, microservices, en-US]
permalink: /posts/hacking-moleculer-and-building-pyleculer-ml-ai-microservices
---

Pylecular started as a genuine experiment. I wanted to combine my background in distributed systems with the lessons I had learned working with [Moleculer.js](https://moleculer.services/) a few years back. The goal wasn’t to build a framework or replace anything, it was to see what would happen if I took a powerful, event-driven microservice model and blended it with Python’s machine learning ecosystem.

> Check Pylecular at: [https://github.com/alvaroinckot/pylecular](https://github.com/alvaroinckot/pylecular)

Python is already rich with ML tools. What it often lacks is a simple, expressive way to plug those tools into a distributed architecture. I was curious: could I take the things Moleculer does so well: brokers, actions, events, services; and apply them in a Pythonic way, letting models and data flows behave like modular agents? Could I create something that made it easier to spin up small, reactive ML systems, without the ceremony of setting up an entire platform?

I wasn’t trying to solve every problem. I just wanted to connect what I already knew and see how far I could push it.

Moleculer’s core concept is clean and effective: services register actions, and those actions can be called across the system via a central broker. No HTTP routes, no external APIs, just names and messages.

Like this:

```js
broker.createService({
  name: "math",
  actions: {
    add(ctx) {
      return ctx.params.a + ctx.params.b;
    }
  }
});
```

And then:

```js
broker.call("math.add", { a: 5, b: 3 });
```

Simple. Modular. Event aware by default.

This wasn’t just about reimplementing a concept in Python. One of the design goals is full interoperability with existing Moleculer services written in Node.js. That means a service written in Python using Pylecular can call actions or emit events to a Node.js Moleculer service, and vice versa. They speak the same protocol. They use the same service and action structure.

This is important because the Moleculer ecosystem is already rich. It has proven patterns for service discovery, load balancing, fault tolerance, and more. There is a whole world of tooling built around it, from API gateways to metrics to transporters like NATS or Redis. With Pylecular, you do not have to start from scratch. You can build Python native ML services that plug directly into that ecosystem and start benefiting from it immediately.

At this point, I wasn’t trying to innovate. I was trying to learn. Rebuild the mechanism. Understand the gears.

Here’s what a Pylecular service looks like, inspired by Moleculer’s simplicity:

```python

from pylecular.service import Service
from pylecular.decorators import action

class MathService(Service):
    name = "math"

    @action(params=["a", "b"])
    async def add(self, ctx):
        return ctx.params["a"] + ctx.params["b"]
```

And the call:

```python
result = await broker.call("math.add", {"a": 5, "b": 3})
```

It worked.

Once the core pattern was there, I started thinking about what else it could do.

That’s when I wrapped a model.

Just a basic LinearRegression. I gave it a predict method, exposed that as an action, and registered it with the broker.

```python
from sklearn.linear_model import LinearRegression
import numpy as np

class MLService(Service):
    name = "ml"

    def __init__(self):
        super().__init__(self.name)
        X = np.array([[1], [2], [3], [4]])
        y = np.array([2, 4, 6, 8])
        self.model = LinearRegression().fit(X, y)

    @action(params=["x"])
    async def predict(self, ctx):
        x = float(ctx.params["x"])
        return float(self.model.predict([[x]])[0])
```

Now, any part of the system could say: “Hey, ml.predict, what’s your answer for x equals 4?”

No HTTP. No Flask. No REST. Just call the model.

And then I thought, what if it could also listen?

In Moleculer, and now in Pylecular, services don’t just respond. They listen. They emit. They react.

So I added events.

Imagine this: when a model finishes training, it emits model.trained. Another service, maybe a dashboard or a cache invalidator, is subscribed to that. It reacts in real time.

In Pylecular, that looks like:

```python
from pylecular.decorators import event

@event("model.trained")
async def handle_trained(ctx):
    print("New model trained. Refreshing cache...")
```

And just like that, your system becomes alive.

Models aren’t just endpoints. They’re agents. They participate in workflows, listen to signals, respond to change.

This is what excited me. This was the architecture I wanted.

Serving ML models isn’t just about exposing a function. It’s about integrating into a system that can adapt, scale, and respond in real time. And doing it in Python, where most ML tooling already lives, makes the entire process feel native.

With this foundation, I began to imagine what Pylecular could be.

It’s not trying to be the next ML platform. It’s not even trying to be the next FastAPI or Ray Serve. What it does is make it trivially easy to treat a model as a service. And that service becomes part of a larger network of components, data ingestors, trainers, notifiers, loggers, dashboards, all communicating through actions and events.

Suddenly, spinning up a minimal ML platform isn’t a two month project. It’s a matter of defining a few services, connecting them through the broker, and letting events flow.

That modularity accelerates innovation. You can prototype faster, test new ideas, and integrate external tools easily. Want to swap one model for another? Just change the service. Want to broadcast that a new dataset is available? Emit an event. Everything stays decoupled and composable.

This aligns closely with a modern platform strategy: accelerating innovation through modularity, enabling reuse and leveraging across teams, and pushing routine operations like predictions into the realm of commoditization, something cheap, fast, and reliable.

And here’s where it gets even more interesting.

Pylecular naturally aligns with the goals of the [Model Context Protocol](https://modelcontextprotocol.io/introduction). With its Moleculer inspired service registry and event system, services can be dynamically discovered and composed. Tools, models, and workflows become addressable pieces within a shared protocol space. This opens the door for true composability, where each ML component is not just a script or container but a named, living service in the context of a wider system.

There are mature and powerful tools out there" Kubeflow, KServe, LangChain, and other. They are designed for robustness, scalability, and reliability. They’re essential when the stakes are high and the systems are complex.

But what about the hackers? The builders who want to spin something up tonight just to see if it works? The people who learn by stress testing ideas and connecting small pieces to see what breaks and what surprises them?

That’s what Pylecular is for. It’s a playground, not a platform. A way to stretch the tools I already have, to see how far I can take a simple broker, a model, and some events.

It’s early. It’s messy. But it’s fun.

> Models as a service, systems as conversations.

And it’s open source. [***Check it out***](https://github.com/alvaroinckot/pylecular). Play with it. Hack it. Break it.
