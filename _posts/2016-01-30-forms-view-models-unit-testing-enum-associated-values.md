---
layout: post
title: Making Forms Coding A Little More Fun with View Models, Unit Testing, and Enumerations with Associated Values
permalink: making-forms-coding-a-little-more-fun-with-view-models-unit-testing-and-enumerations-with-associated-values
comments: true
---

Enumerations have much [more power](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Enumerations.html) in Swift than in C. For one they are not restricted to integer type, e.g. they can be created with "raw" String value and still be matched in `switch` expression. They can even contain associated values! Canonical example here is `Result`, used for error handling.

~~~swift
enum Result<T, Error: ErrorType> {
    case OK(T)
    case Error(Error)
}
~~~

Let's look further than that. Associated value data can be of any type: struct, class, enum or even closure. This opens great possibility for passing around configurations with enum cases.

When we have an app with lots of input forms, implementing them gets tedious very quickly. We really want to streamline the process. There are [a bunch](https://github.com/xmartlabs/XLForm) of [libraries dealing](https://github.com/hyperoslo/Form) with [this](https://github.com/venmo/Static). However, such big universal solutions come with a cost. It includes time spent familiarizing with framework's intricacies, running into limitations and working around them. Along the way some needed feature may just not be there. Fork time. All of this can be more effort than building solution that matches our specific needs. [Tradeoffs](https://twitter.com/robertoguerra19/status/539901289317269505), right?

As an alternative, below is a set of simple guidelines that work both for dealing with static and dynamic collections of items. To keep example brief, it deals only with a few cell types: one for text input, one for selection from group of options, and one for boolean values.

<!--more-->

~~~swift
enum ItemCellType {
    case Input(title: String, value: String?, placeholder: String?, inputType: InputType, onInput: InputHandler),
    Selection(title: String, value: () -> String?, options: [String], onSelection: SelectionHandler)
    Switch(title: String, value: Bool?, onValueChange: SwitchHandler)
}
~~~

Based on that we can implement a template View Controller (or Factory) producing cells from `ItemCellType` and its configurations. With this at hand, task of creating _Yet Another Form_ becomes much easier: just configure `ItemCellType`s and feed them right to the template View Controller. Plus, when new item type is added, if we played our cards right, compiler will let us know about `switch` that needs to be made exhaustive.

So how to feed those configurations then? Sections of `ItemCellType`s can reside inside of a View Model. Once sections are set up, View Controller can delegate data source methods to View Model via interface like this:

~~~swift
protocol FormViewModelType {
    func numberOfSections() -> Int
    func numberOfItemsInSection(section: Int) -> Int
    func itemCellTypeAtIndexPath(indexPath: NSIndexPath) -> ItemCellType
}
~~~

Notice there are no references to UIKit in this protocol. Keeping references to `UIView` classes from View Model, means views can be switched out a little bit easier. Cell implementantions can be replaced at will, without touching logic that drives form. As developers we want to [embrace change](https://vimeo.com/140421667) and changing logic coupled with presentation is a real pain (and vice versa). Having such ViewController/ViewModel structure enables us to just create another View Controller that communicates with `FormViewModelType`. Extra bonus points for User Interface A/B testing possibilities: using protocol allows to iterate over different types of interfaces without doing Big Rewrites. View Model can be used to drive `UITableView` as well as `UICollectionView`.

Implementations of `FormViewModelType` can contain specific code for model updates, validation, passing data to services via [gateway](http://blog.cleancoder.com/uncle-bob/2016/01/04/ALittleArchitecture.html) (be it REST API or local storage), etc.

Closures that handle input are setup in View Model that has access to Model. Thanks to this reading from/writing to Model is "local" and additional mapping between Model properties and index paths can be avoided - it's especially helpful in case when static form is long and has many different types of input.

Input handling closures also make automated testing of form interaction logic way easier. Does this section collapse when we flip this switch? We just need right `ItemCellType` and interact with it's handler.

~~~swift
func test_extraDataSectionToggle_SwitchOff_ExtraDataSectionIsEmpty {
    let vm: ConcreteFormViewModel = viewModel()

    // act
    let switchSection = ConcreteFormViewModel.SectionIndex.ExtraDataSwitch
    let extraDataIndexPath = NSIndexPath(atRow: 0, inSection: switchSection)
    let item = vm.itemCellTypeAtIndexPath(extraDataIndexPath)
    guard case .Switch(let config) = item else {
        XCTFail("\"\(item)\" does not match ItemCellType.Switch")
    }    
    config.onValueChange(false)

    let inputSection = ConcreteFormViewModel.SectionIndex.ExtraDataInput
    let result = vm.numberOfItemsInSection(inputSection)
    XCTAssertEqual(0, result)
}
~~~

Next step is to get View Models's behaviour for "on" state under test. "Act" part of the above test is kinda ugly and obfuscates a little what is actually going on. Similar interaction is needed for the test we're writing next. Looks like a case of [structural duplication](http://mr-v.github.io/custom-error-assertions-in-swift/). We have a passing test, so its structure can be refactored without worrying too much - if something breaks in the process, we'll know right away. Let's extract "act" part to separate method and abstract tapping Extra Data Section Switch to make tests more maintainable. See next test with improved "act" part:

~~~swift
func test_extraDataSectionToggle_SwitchOn_ExtraDataSectionHasThreeItems {
    let vm = viewModel()

    // act
    changeExtraDataSectionSwitch(value: true)

    let section = ConcreteFormViewModel.SectionIndex.ExtraDataInput
    let result = vm.numberOfItemsInSection(section)
    XCTAssertEqual(3, result)
}

// MARK: -

private func changeExtraDataSectionSwitch(value value: Bool) {
    let switchSection = ConcreteFormViewModel.SectionIndex.ExtraDataSwitch
    let extraDataIndexPath = NSIndexPath(atRow: 0, inSection: switchSection)
    let item = vm.itemCellTypeAtIndexPath(extraDataIndexPath)
    guard case .Switch(let config) = item else {
        XCTFail("\"\(item)\" does not match ItemCellType.Switch", file: __FILE__, line: __LINE__)
    }    
    config.onValueChange(value)
}
~~~

With this ViewModel setup all interactions and form's "view data" (e.g. titles) can be unit tested without resorting to brittle UI testing. Does this selection contain all of the required options? What happens to model if we pick something? Similar with testing validations or checking that Model passes through the gateway with right configuration, etc. We can get pretty far with `ItemCellType` and no `UIView` in sight!

Above approach is pretty straightforward, but gives that extra flexibility we need as developers. I'm attaching no source code, since each app have different set of requirements and handles many details differently.

Using View Models with Swift's enumerations to configure View/View Controller:

- enables straightforward change of views, even entire controllers, without touching underlying logic,
- is great for A/B testing,
- makes automated testing easier,
- promotes creating visual components, which leads to having more consistent UI across the app.
