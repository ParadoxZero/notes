That said, let's dive into the topic of MVVM by means of an example. MVVM guides us how to distribute responsibilities between classes in a GUI application (or between layers - more about this later), with the goal of having a small number of classes, while keeping the number of responsibilities per class small and well defined.

'Proper' MVVM assumes at least a moderately complex application, which deals with data it gets from "somewhere". It may get the data from a database, a file, a web service, or from a myriad of other sources.

## Example

In our example, we have 2 classes View and Model, but no ViewModel. The Model wraps a csv-file which it reads on startup and saves when the application shuts down, with all changes the user made to the data. The View is a Window class that displays the data from the Model in a table and lets the user edit the data. The csv content might look somewhat like this:

```
ID, Name,    Price   
1,  Stick,   5$   
2,  Big Box, 10$   
3,  Wheel,   20$   
4,  Bottle,  3$
````

### New Requirements: Show price in Euro

Now we are asked to make a change to our application. The data consists of a 2-dimensional grid which already has a "price" column, containing a price in USD. We need to add a new column which shows prices in Euro in addition to those in USD, based on a predefined exchange rate. The format of the csv-file must not change because other applications work with the same file, and these other applications are not under our control.

A possible solution is to simply add the new column to the Model class. This isn't the best solution, because the Model saves all the data it exposes to the csv - and we do not want a new Euro price column in the csv. So the change to the Model would be non-trivial, and it would also be harder to describe what the Model class does, which is a [code smell](http://en.wikipedia.org/wiki/Code_smell).

We could also make the change in the View, but our current application uses data binding to display the data directly as provided by our Model class. Because our GUI framework doesn't allow us to introduce an additional calculated column in a table when the table is data bound to a data source, we would need to make a significant change to the View to make this work, making the View a lot more complex.

### Introducing the ViewModel

There is no ViewModel in the application because until now the Model presents the data in exactly the way the Csv needs it, which is also the way the View needed it. Having a ViewModel between would have been added complexity without purpose. But now that the Model no longer presents the data in the way the View needs it, we write a ViewModel. The ViewModel projects the data of the Model in such a way that the View can be simple. Previously the View class subscribed to the Model class. Now the new ViewModel class subscribes to the Model class, and exposes the Model's data to the View - with an extra column displaying the price in Euros. The View no longer knows the Model, it now only knows the ViewModel, which from the point of the View looks the same as the Model did before - except that the exposed data contains a new read only column.

### New requirements: different way to format the data

The next customer request is that we should not display the data as rows in a table, but instead display the information of each item (a.k.a. row) as a card/box, and display 20 boxes on the screen in a 4x5 grid, showing 20 boxes at a time. Because we kept the logic of the View simple, we simply replace the View entirely with a new class that does as the customer desires. Of course there is another customer who preferred the old View, so we now need to support both. Because all of the common business logic already happens to be in the ViewModel that is not much of an issue. So we can solve this by renaming the View class into TableView, and writing a new CardView class that shows the data in a card format. We will also have to write some glue code, which might be a oneliner in the startup function.

### New requirements: dynamic exchange rate

The next customer request is that we pull the exchange rate from the internet, rather than using a predefined exchange rate. This is the point where we revisit my earlier statement about "layers". We don't change our Model class to provide an exchange rate. Instead we write (or find) a completely independent additional class that provides the exchange rate. That new class becomes part of the model layer, and our ViewModel consolidates the information of the csv-Model and the exchange-rate-Model, which it then presents to the View. For this change the old Model class and the View class do not even have to be touched. Well, we do need to rename the Model class to CsvModel and we call the new class ExchangeRateModel.

If we hadn't introduced the ViewModel when we did but had instead waited until now to do so, the amount of work to introduce the ViewModel now would be higher because we need to remove significant amounts of functionality from both of the View and the Model and move the functionality into the ViewModel.

Source: [https://stackoverflow.com/a/20524980/4929982](https://stackoverflow.com/a/20524980/4929982)