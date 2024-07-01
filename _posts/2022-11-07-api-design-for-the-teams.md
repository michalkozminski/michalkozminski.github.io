---
layout:     post
title:      API design for the teams
date:       2022-11-07
summary:    Practical advice for developing distributed APIs
categories: API
thumbnail: API
tags:
 - API
 - API design
 - Team Management
 - Software Development
---
When designing a Building consistent API can be difficult as teams grow and requirements get more complex. That can lead to cycle where expanding the team doesn’t make development easier. In this article I’m going to walk you though some of practices which mitigate complexity and allow building good consistent API even when your requirements and teams grow.

Web apps used to host HTML directly from the server. Making any changes in such a model were quite simple since the only API we provided were URLs that users could access. And the only expectation was that */products/123* will always take them to the same product. UI and workflows could change to a certain degree without users being affected. Our client would only expect to always find the same product under this URL. The only contract we have with our clients is URL.

Nowadays when developing applications we have to think about our API layer. We might have multiple native apps as well as a web app and some third-party integrations. This makes the development of applications highly depend on well-defined API.

![https://miro.medium.com/v2/resize:fit:1400/1*xFO7lUy9Fwi2j2bXqkW5lQ.png](https://miro.medium.com/v2/resize:fit:1400/1*xFO7lUy9Fwi2j2bXqkW5lQ.png)

So how do we approach this problem? You probably hear more and more about different approaches to designing applications, like mobile-first, API first, and web first. But which one is right for you?

## **Start with use-case in mind**

Whenever defining API, you should start with fundamental use-case analysis. To avoid getting lost in an endless loop of questioning everything, here are good starting questions:

- Who is going to consume it?
- What is its purpose?
- How reliable should it be?

I recommend first understanding if that API is going to be used by third parties you can’t influence (it?). API depreciation and versioning is expensive so it’s worth spending extra time at an early stage. In this case that means focusing on making your interface more robust and extensible.

After answering that question start with drawing your requirements. To make it easier we can use table notation for it. On the top, we start with requirements and then we split them into Functional and non-functional.

![https://miro.medium.com/v2/resize:fit:1400/1*ho7xa9-P2v5kIoanB0xq3A.png](https://miro.medium.com/v2/resize:fit:1400/1*ho7xa9-P2v5kIoanB0xq3A.png)

This makes our basic functional and non-functional requirements straightforward without bringing too much clutter. A key point in working in teams is making your decision process as transparent as you can. After pointing out our most important topics, let’s start addressing them.

## **Using tools to supercharge your process**

To make API design iterations faster we’re going to use some tools. In case you’re designing synchronous API we can look at OpenAPI or AsyncAPI in case it’s asynchronous. In our case, we look at a fully synchronous API.

We start by drawing how we imagine our first use-case *getting all employees* implemented.

We know it will be used by the internal app to display all employees listed as well as third-party integration which will be interested in one-way synchronization. Let’s start sketching our API. At this stage we want to create minimal implementation which can help us set the direction.

![https://miro.medium.com/v2/resize:fit:1400/1*eRrIXplHvdT7U0umUBNWnw.png](https://miro.medium.com/v2/resize:fit:1400/1*eRrIXplHvdT7U0umUBNWnw.png)

[https://gist.github.com/michalkozminski/a0a900d2d9a0ccc94caba36077d662e5](https://gist.github.com/michalkozminski/a0a900d2d9a0ccc94caba36077d662e5)

This is the first version of API, it does indeed return a list of employees. So we can check one box off of our list. Also gives you a better look on possible problems we could run into during development.

You can start to see why to begin with the documentation. It allows you to  spot problems as early as possible. The app will have to fetch everything all the time so our peak performance might suffer as well as the app experience. Let’s start addressing these challenges one by one. We start with data fetching. Let’s add an option to paginate our results. Allowing us to control how much data is sent to the client at once. This is a breaking change as our response is not an array anymore but an object. It also gives us more flexibility on later changes.

![https://miro.medium.com/v2/resize:fit:1400/1*q19HCY2QCrtQIhG1s_e6hA.png](https://miro.medium.com/v2/resize:fit:1400/1*q19HCY2QCrtQIhG1s_e6hA.png)

[https://gist.github.com/michalkozminski/8abead80d3db9aedb78ba98765b3214c](https://gist.github.com/michalkozminski/8abead80d3db9aedb78ba98765b3214c)

Now our API supports pagination. With this documentation we can create a running mock server using OpenAPI Generator. This server can be already shared with the team which will be our consumer to gather initial feedback. We can achieve it by using:

```
openapi-generator generate -i employees_service.yaml -g kotlin
```

This will create Kolin service which can be deployed and allow us rapid prototyping .

As you can see so far we spend only a brief amount of time ending with a very basic example that works. Using OpenAPI allows you to communicate quickly with the team and get feedback instantly.

## **Shift Left**

![https://miro.medium.com/v2/resize:fit:1400/1*fMs0v-UTuDwN5W2-AamzeA.png](https://miro.medium.com/v2/resize:fit:1400/1*fMs0v-UTuDwN5W2-AamzeA.png)

The purpose of this example is to show the shift-left mentality in API design. Now when we have a very basic API we can start to incorporate feedback from everyone. So who should be involved?

- API committee (if your company has such)
- Future consumers of your API
- Security team
- Performance/QA teams (if your company has such)

Let’s imagine we send this API and our API committee comes back with some advice:

- For content type we should use ***application/vnd.company.{subdomain}.{version}+{format}***
- We didn’t define error cases
- We have absent caching

Future consumer:

- they are most interested in new employees to synchronise it

Our security team:

- We need to know who is sending requests

And our performance team pointed out that:

- Employees don’t change that often and pulling data will cost a lot

**Save time and money**

Allowing early team collaboration can save a lot of resources. It takes over 30x more time and effort to address a problem after it’s been deployed. So we can imagine that all the issues we found would take a tremendous amount of time if not for the fact we made the feedback loop shorter.

![https://miro.medium.com/v2/resize:fit:1400/1*PHkCgOWcRW3QuBebVgM8Fw.png](https://miro.medium.com/v2/resize:fit:1400/1*PHkCgOWcRW3QuBebVgM8Fw.png)

**Addressing comments**

Let’s try to address issues raised by the committee. We will update our OpenAPI and get feedback from them. As you can see below, we addressed problems raised by the team. Now we can share it again with the team which will consume it to get their view on caching as well as the errors we provided.

![https://miro.medium.com/v2/resize:fit:1400/1*TjrAm5IhlFf0rF8Tb18zfg.png](https://miro.medium.com/v2/resize:fit:1400/1*TjrAm5IhlFf0rF8Tb18zfg.png)

[https://gist.github.com/michalkozminski/dd9dc8d9503590b35edb323537b06485](https://gist.github.com/michalkozminski/dd9dc8d9503590b35edb323537b06485)

Here we can see the updated OpenAPI

By again using a code generator, we can generate an updated version for the consumer team to experiment with it.

## **Keeping short loop**

Building consistent API in organizations requires constant evaluation. The process you create should empower others and make their work easier. Engineers shouldn’t feel that they have to spend too much time on designing API. Establish a process that helps them and doesn’t create friction.

![https://miro.medium.com/v2/resize:fit:1400/1*vNR51tjcDvshVKjUJq5wQg.png](https://miro.medium.com/v2/resize:fit:1400/1*vNR51tjcDvshVKjUJq5wQg.png)

## **How to create an easy process**

Start with forming one team which will be advising others on matters of design. This team’s main responsibility is to keep API consistent and developer-friendly. To achieve that they **can use** multiple tools like

- API specs
- Global linting of API docs
- Training
- Advises per case based
- Delegation poker

Let’s look at each of these and see how we could make it easy

## **API specs**

These should be straightforward documents. You want to make sure people will follow it so start with a short spec explaining the most important things like naming convention, auth, versioning, entity representation, and error handling. You can base your spec on already existing ones like:

[Zalando](https://github.com/zalando/restful-api-guidelines), [Google](https://cloud.google.com/apis/design)

## **Advising**

The committee helps teams answer the questions. Based on the questions brought to the team they keep specs up to the date and refer questions to already existing solutions.

## **Global Linting of API docs**

To make sure teams follow basing rules you could enforce the linting of OpenAPI. Basic errors like lack of authentication or wrong naming using CamelCase instead of underscore might be caught without human intervention.

## **Training**

This is one of the most powerful strategies. Make sure to educate your engineers by showing case-studies. Each successful implementation should be a reason to celebrate it and should also help shape best practices in your company.

## **Start today**

I hope I managed to convince you to work on your processes and include technology in them. This might look like starting iteratively. Start small and see how process and technology improve your software. Remember that your goal is empowering the engineering team and help them build better software. The process you put in place should be centered around engineers, not API design. Get as much feedback as possible and improve process step by step.

Note post cross-published: https://medium.com/beekeeper-technology-blog/api-design-for-the-teams-afa63636906b