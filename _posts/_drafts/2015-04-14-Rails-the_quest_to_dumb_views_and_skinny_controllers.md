---
layout: post
title: "Rails: The quest for dumb views and skinny controllers"
description: "There is no one who loves pain itself, who seeks after it and wants to have it, simply because it is pain..."
comments: true
keywords: "rails, helpers, presenters, views, controllers"
---

_These are my notes for a talk i did on Ruby.do on 2015, you can find the slides here: [slides.com/vjustov/decorators](https://slides.com/vjustov/decorators#/20)_

The Model-View-Controller design pattern, as used by Ruby on Rails and other web application frameworks, has clear separations: each part has its own role to play. Views represent your HTML templates, Model represent your business objects, and controllers handle your requests. But sometimes it can leave us with a bit of a gap.

## Helpers

I assume that many of you have written (and are still writing) helpers for your views in your awesome Rails projects. A helper, as you all know, is a module which contains methods that are vital for the simplification and the dryness of your views.

In Rails, a helper is a function. Like  the #capitalize method that helps me capitalizing the user name, which is freaking awesome. As this is pretty simple behaviour, let's call helpers like this utility methods. They modify the input parameter, compute something or escape strings. Pretty straight-forward. This helper will iterate through news items for a particular user and render markup, maybe using several partials, Since it actively renders templates, let's call this a view component.

And because helpers are meant to be used within your views they often contain html code (either plain or ruby code that produces HTML) which, as you may have noticed, is not sery flexible.

Using the #capitalize methods happens without a receiver. The method is globally available in the view since Rails somehow mixes the helper into the view. So, what happens if I have two #capitalize methods in two different helpers mixed in the same view? I don't have a clue. Do you?

There is the helpers directory. You can create your own modules there or use the default one: ApplicationHelper, which should exist in this directory. Let's create your own module. As you see helpers are mixed with views by Rails (in the background) and their methods are available in Views. To avoid cluttering the templates with boilerplate code, a number of helper classes provide common behavior for forms, dates, and strings. It's also easy to add new helpers to yur application as it evolves.

I recently wanted to use helpers in a way that would allow helpers defined later overwrite methods defined by helpers included earlier (that's an expected behavior I believe). It was total pain in the ass for two reasons:
a) rails by default loads all helpers for every controller
b) the helper which is named after controller is included into view context first.

When I started using Rails years ago I found helpers extremely cool. I could call a method in a view and it would help me by doing something The method was simply there, no need to  worry about its source and how to access it, just call it

Helpers in Rails are modules, which do not allow inheritance. If l'd need a foreign method I'd have to include another module into the helper module.

Why is the receiver magically the Action View instance and not the controller when using helpers?

In my experience helpers in Rails apps tend to devolve into large, disorganized bags of unrelated methods. Often these methods repeat the same conditional business logic over and over ito again.

## Decorators

Render the In object-oriented programming, the decorator pattern (also known as Wrapper, an alternative naming shared with the Adapter pattern) is a design pattern that allows behavior to be added to an individual object, either statically or dynamically, without affecting the behavior of other objects from the same class. The decorator pattern çan be used to extend (decorate) the functionality of a certain object statically, or in some cases at run-time, independently of other instances of the same class, provided some groundwork is done at design time. This is achieved by designing a ntw decorator class that wraps the original class. This wrapping could be achieved by the following sequence of steps: extend

## Martin Fowler

The essence of a Presentation Model is of a fully self-contained class that represents all the data and behavior of the UI window, but without any of the controls used to render that UI on the screen. A view then simply projects the state of the presentation model onto the glass.

To do this the Presentation Model will have data fields for all the dynamic information of the view. This won't just include the contents of controls, but also things like whether or not they are enabled. In general the Presentation Model does not need to hold all of this control state (which would be lot) but any state that may change during the interaction of the user. So if a field is always enabled, there won't be extra data for its state in the Presentation Model.

## Presenter

The Presenter pattern addresses bloated controllers and views containing logic in concert by creating a class representation of the state of the view. An architecture that uses the Presenter pattern provides view specific data as attributes of an instance of the Presenter The Presenter's state is an aggregation of model and user entered data.

The Presenter, as we use it, knows what the View needs, and using information from the NModel, prepares this information forthe View. The Presenter is created by the Controller, initialized with the Model, and passed to the View: The View in turn calls Presenter methods

the Controller constructs the Presenter object and passes the Model instance to it and makes the Presenter available to the view.

Therefore the Presenter knows about the Model, performs calculations on the data in the Model object and prepares the information to be used by the View. The View's only concern is the structure of the elements,

the Rather then sprinkling these presentation-oriented decisions and knowledge across the whole MVC architecture the Presentation Object drops in as an intermediary between the view and the model. It is responsible for knowing that the logic it contains will be presentation logic. This makes it the perfect place to put logic that otherwise shows up in the model, contoller or view.

## Exhibit

Another that object satisfy our need for an object which mates a model and a view, its the Exhibit object. If the Model is concerned with storing and manipulating business data, and the View is concerned with displaying it, you can think of the Exhibit as standing between them deciding which data to show, and how to show it. It may also provide some extra presentation-specific information (such as the specific URLS for related resources) which the business model has no knowledge of by itself.

The Exhibit object is so named because it is like a museum display case for an artifact or a work of art. It does not obscure any of the features of the object being presented. Rather, it tries

showcase the object in the best liglit to a human audience, while also presenting meta information about the object and eross-references to other objects in the museum's collection.

Technically, exhibit objects are a type of Decorator specialized for presenting models to an end user. In fact, I briefly considered calling then "Presenter Decorators", but tat term in a bit umwieldy, as well as being a litile too easy to confuse with other "Presenter" terms (on which more later).

So, exhibits are just decorators, Often with a decorator, you'll want to do more than just forward methods onto the delegate object. You'll likely want to add some additional functionality. Such is the case with exhibits. The additional fiunctionality added will extend (but not disrupt) the delegate object.

The primary goal of exhibits is to connect a model object with a context for which it's rendered. This means it's the exhibit's responbility to call the context's rendering method with the model as an argument. In most Rails applications, the "context" will be the view or the controller.

The exhibhit could also take the responsibility of attaching metadata to an object. In the following example, we'll attach some additional information about an exhibit which is more akin to the model attributes than how the model is rendered.

A key differentiator between exhibits and presenters is the language they speak.

The purpose of this exhibit is to provide metadata (the #additional_info method) as well as call the context's #render method (within the #render method). We're defining two different "contexts", a text environment and a browser environment.

## Exhibit + Presenter

magine we rated a don'tfarelhow miany dor fe chaas long as it can render itself property make Exhibit + Prcsenter Reference fr helpers flauy

Presenters and exhibits are not mutually exclusive. We can combine these two concepts to create a presenter which contains view-related logic and knoWs how to render itself or its components.

In this example, we combine presenters and exhibits to take advantage of both. We use presenters as a representation of the object in the view and to deal with any view-related logic. We use exhibits to manage rendering the objects.

When we tease these two concepts of "Presenter" and "Exhibit" into separate entities, we realize that they are complementary patterns that could casily work together in an application. This text deals mainly with Exhibits, but it's easy to imagie an app in which Exhibits are aggregated together under a combined Presenter för a particular.

## So what about what i read this morning

In the above examples, where presenters and exhibits are used in conjunction, we've demonstrated the true power of decorators. Calling CarPresenternewCar.newi decorates the new car twice, once with an exhibit and once with a presenter. The beauty, however, is that we can treat the decorated car exactly as we would treat a normal car Since it's a true decorator (it delegates all unrecognized methods to the delegate object), we can treat it as though it were a car. For instance, the following works:

Decorators are a form of object composition. We can fashion complex objects with composition instead of inheritance, often a desired technique. Keep in mind, however, that composition obfuscates the identity of the delegate object. To get composition to work correctly with Rails requires tricking Ruby into thinking the decorated object is, in fact, the underlying delegate object.
