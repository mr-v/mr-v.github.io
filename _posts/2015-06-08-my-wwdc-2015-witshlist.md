---
layout: post
title: My WWDC 2015 Wishlist
permalink: my-wwdc-2015-wishlist
comments: true
---
WWDC is about to start in 9 hours. Here's my modest wishlist:

- stable tools and iOS - I know, boring,
- nice refactoring tools in Xcode - if open source Flashdevelop can do it, Apple can do it!
- extensions to Swift language:
 - `Result` enum - everybody have been blogging and talking about it for the last year, plus Rust has [it](http://doc.rust-lang.org/std/result/enum.Result.html),
 - `await` and `async` keywords - we are closely catching up on JavaScript's 10+ different ways of doing async stuff. It's such a big part of our work, that support for it should be baked into the language/compiler. See how nice it is in [C#](https://msdn.microsoft.com/en-us/library/hh191443.aspx):
 
 ~~~c#
  public partial class MainWindow : Window
  {
    // Mark the event handler with async so you can use await in it. 
    private async void StartButton_Click(object sender, RoutedEventArgs e)
    {
        // Call and await separately. 
        //Task<int> getLengthTask = AccessTheWebAsync(); 
        //// You can do independent work here. 
        //int contentLength = await getLengthTask; 

        int contentLength = await AccessTheWebAsync();

        resultsTextBox.Text +=
            String.Format("\r\nLength of the downloaded string: {0}.\r\n", contentLength);
    }


    // Three things to note in the signature: 
    //  - The method has an async modifier.  
    //  - The return type is Task or Task<T>. (See "Return Types" section.)
    //    Here, it is Task<int> because the return statement returns an integer. 
    //  - The method name ends in "Async."
    async Task<int> AccessTheWebAsync()
    { 
        // You need to add a reference to System.Net.Http to declare client.
        HttpClient client = new HttpClient();

        // GetStringAsync returns a Task<string>. That means that when you await the 
        // task you'll get a string (urlContents).
        Task<string> getStringTask = client.GetStringAsync("http://msdn.microsoft.com");

        // You can do work here that doesn't rely on the string from GetStringAsync.
        DoIndependentWork();

        // The await operator suspends AccessTheWebAsync. 
        //  - AccessTheWebAsync can't continue until getStringTask is complete. 
        //  - Meanwhile, control returns to the caller of AccessTheWebAsync. 
        //  - Control resumes here when getStringTask is complete.  
        //  - The await operator then retrieves the string result from getStringTask. 
        string urlContents = await getStringTask;

        // The return statement specifies an integer result. 
        // Any methods that are awaiting AccessTheWebAsync retrieve the length value. 
        return urlContents.Length;
    }

    void DoIndependentWork()
    {
        resultsTextBox.Text += "Working . . . . . . .\r\n";
    }
}
 ~~~

 List may be short, but those features are essential and would make our life easier. With them in place we could place more focus on solving interesting problems.
