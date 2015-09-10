This year’s Google IO introduces an awesome new framework for Android developers that allows for “binding” of views to fields on an arbitrary object. When a field is updated, the framework is notified and the view is updated automatically.

This system is extremely powerful, and will enable us to use a development pattern that’s been used in the Windows world for some time called Model-View-ViewModel (MVVM). Before we jump into any code, it’s important to understand the basic concepts of this architecture, and how it can benefit you and your app.
The MVVM pattern consists of three parts:
Model – Represents your business logic
View – Displayed content
ViewModel – The object that ties those two together
A ViewModel interface provides two things: actions and data. Actions change the underlying model (click listeners, text changed listeners, etc.), and data is the content of that model.
For example, a ViewModel for an auction page might expose as data an image of the item, a title, description, and price. It might expose as actions the ability to bid, buy, or contact the seller.
In traditional Android architecture, the controller pushes data to the view. You find the view in your Activity, then set content on it. With MVVM, your ViewModel alters some content and notifies the binding framework that content has changed. The framework will then automatically update any views bound to that content. The two components are only loosely coupled through that interface of data and commands.
What makes this pattern so awesome, aside from the obvious conveniences of a “smarter” view, is that it lends itself to testing so well.
Because a ViewModel does not depend on the View anymore, you can test a ViewModel without a View even existing. With proper dependency injection for other dependencies, it is very straightforward to test.
For example, instead of binding a VM to a real view, one might create a VM in a test case, give it some data, then call actions on it, to make sure the data is transformed properly. In the auction example, you might create a VM with a mock API service, have it load an arbitrary item and bid on it, and ensure that the resulting exposed data is correct. All of this can be done without having to interact with an actual View.
When testing a view, one could create a number of mock view models that expose pre-set data (as opposed to relying on network calls or database contents), and see how that view reacts to any number of data sets.
Hopefully this has given you a better understanding of what the MVVM pattern is, and why it’s useful. Later on, we’ll have some posts that go into the details of implementing this pattern, as well as some other helpful tips and tricks when working with the binding framework.