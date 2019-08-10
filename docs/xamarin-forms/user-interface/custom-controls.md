---
title: "Create a custom control with Xamarin.Forms"
description: "Buid a custom control using Xamarin.Forms, without a custom renderer."
ms.prod: xamarin
ms.assetid: 1689718E-5508-40E9-96A5-04D0C00BEF81
ms.technology: xamarin-forms
author: mshwf
ms.author: crdun
ms.date: 05/23/2019
---
# Create a custom control with Xamarin.Forms

The process of building UI elements requires, at some point, some customizations to give the unique feel and look to the application and to extend the functionality of existing controls. Whether the customization is just overriding the default `TextColor` of the `Entry` or creating a brand new control with new look and behavior, building custom controls can provide the simple solution to achieve that, without the need of custom renderers.

The process for creating custom controls is as follows:

1. Create a subclass of the view you want to extend or modify.
2. Alter the functionality of the subclass by overriding the default value of the base class’s bindable properties and/or create new bindable properties that will interact with user actions.
3. Process inputs through the `propertyChanged` delegate of the newly added bindable properties.

## Create a custom TextSwitch control

The TextSwitch control is an on-off control with text that get highlighted when switched on, see the screenshot below(should look the same on iOS):

[screenshot (on Android)]

The behavior of the control is as follows:

1. The control has on and off states determined by the `IsOn` property.
2. The states are visually distinguished through the `SelectedColor` and `UnselectedColor` bindable properties.
3. When the control is tapped, the selection state is mutated and the text color is updated to the `SelectedColor` or `UnselectedColor`.

Bindable properties is the foundation of custom controls (For more information about Xamarin.Forms bindable properties, see [Xamarin.Forms Bindable Properties
](~/xamarin-forms/xaml/bindable-properties.md))

The steps of creating the TextSwitch control are as follows:
1. Create a subclass from `StackLayout`, name it `TextSwitch`, it holds two children: `Label` and `BoxView`, the following diagram illustrates the control outline:
![](custom-controls-images/togglebutton-layout.png "Togle bar control outline")

When the label is tapped, the selection state is mutated. The visual state is defined by the `TextColor` property of the Label and the `Color` property of the BoxView,

2. Create the bindable properties: `IsOn`, `SelectedColor`, `UnselectedColor`, `Text`, `FontFamily` and `FontSize`. This is the `SelectedColor` property along with the [`BindableProperty`](xref:Xamarin.Forms.BindableProperty) backing field:

```csharp
public static readonly BindableProperty SelectedColorProperty = BindableProperty.Create(nameof(SelectedColor), typeof(Color), typeof(TextSwitch),
defaultValue: default(Color), propertyChanged: CustomPropertyChanged);

 public Color SelectedColor
 {
     get { return (Color)GetValue(SelectedColorProperty); }
     set { SetValue(SelectedColorProperty, value); }
 }
 ```
 
> [!NOTE]
> A bindable property is a special type of property, where the property's value is tracked by the Xamarin.Forms property system. 
> The purpose of bindable properties is to provide a property system that supports data binding, styles, templates, and values set through
> parent-child relationships.
The process of creating a bindable property is as follows:
> 1. Create a [`BindableProperty`](xref:Xamarin.Forms.BindableProperty) instance with one of the [`BindableProperty.Create`](xref:Xamarin.Forms.BindableProperty.Create*) method overloads.
> 2. Define property accessors for the [`BindableProperty`](xref:Xamarin.Forms.BindableProperty) instance.
>   
> For more information about Xamarin.Forms bindable properties, see [Xamarin.Forms Bindable Properties
> ](~/xamarin-forms/xaml/bindable-properties.md)

3. Attach the `propertyChanged` delegate of the bindable properties to `CustomPropertyChanged` method, that will process inputs from the user, like setting the label's `Text` and `TextColor` properties from the `TextSwitch`'s `Text` and `UnselectedColor` properties respectively, and add a `TapGestureRecognizer` to the Label’s `GestureRecognizers` collection that will mutate the selection state of the `TextSwitch` when the label is tapped.

```csharp
private static void CustomPropertyChanged(BindableObject bindable, object oldValue, object newValue)
{
    if (newValue == null) return;
    ((TextSwitch)bindable).Render();
}

private void Render()
{
    button = new Label
    {
        TextColor = UnselectedColor,
        Text = Text,
        BackgroundColor = BackgroundColor,
        FontFamily = FontFamily,
        FontSize = FontSize,
        HorizontalTextAlignment = TextAlignment.Center,
        VerticalTextAlignment = TextAlignment.Center,
        WidthRequest = WidthRequest,
        HeightRequest = HeightRequest,
        Margin = new Thickness(5)
    };

    underLine = new BoxView { HeightRequest = HeightRequest > 0 ? HeightRequest / 10d : 2, Color = BackgroundColor };

    Children.Clear();
    Children.Add(button);
    Children.Add(underLine);
    button.GestureRecognizers.Add(new TapGestureRecognizer()
    {
        Command = new Command(() =>
        {
            IsOn = !IsOn;
            SelectionChanged?.Invoke(this, EventArgs.Empty);
        })
    });


}
```
The `CustomPropertyChanged` is called whenever the bindable property, which [`propertyChanged`](xref:Xamarin.Forms.BindableProperty.BindingPropertyChangedDelegate) delegate is attached to, changes, a change occurs when the property is set when the custom control is initialized. Typically, bindable properties' `propertyChanged` delegate are attached to the same method, when they're responsible for customizing the control's appearance, because we want to make sure all properties are updated once another property changes, for that reason, the `IsOn` bindable property isn't attached to a `propertyChanged` delegate beacuse no UI customization is required based on its initial value, but if the value of the `IsOn` property is set, for example, from data binding, then `propertyChanged` is required to initialize the control with the correct state based on the `IsOn` property value, in this tutorial, the control is always initialized in unselected state (`IsOn` value is `false`).

> [!NOTE]
> When the custom control is initialized, `propertyChanged` delegate is called in the same order as the properties initialization order in XAML (or code), so for properties attached to the same delegate, the last call to the delegate handler is where all properties have been set.

The `Render` method initializes the control properties, for example the `TextColor` property of the label gets the value of `UnselectedColor` property of the custom control beacause the control is rendered in unselected state, similarly, the `BoxView`'s color is initialized with the color of the `BackgroundColor` of the `StackLayout` to hide it, it only gets highlited with `SelectedColor` color when the control is selected. Setting the `WidthRequest` and `HeightRequest` for both the `Label` and `BoxView` ensures they scale with their parent's size.

4. Create `SelectionChanged` event that gets invoked when the label is tapped, and it will notify consumers of the TextSwitch when selection changes:

```csharp
public event EventHandler SelectionChanged;
```

When the label is tapped we need to change the selection state of the control, create `MutateSelect` method and call it in the set accessor of the `IsOn` property that gets mutated when the label is tapped:
```csharp
public bool IsOn
{
    get { return (bool)GetValue(IsOnProperty); }
    set
    {
        SetValue(IsOnProperty, value);
        MutateSelect();
    }
}
```

The `MutateSelect` method is where the selection state gets updated visually when the `IsOn` is mutated:

```csharp
void MutateSelect()
{
    if (IsOn)
    {
        button.TextColor = SelectedColor;
        underLine.Color = SelectedColor;
    }
    else
    {
        button.TextColor = UnselectedColor;
        underLine.Color = BackgroundColor;
    }
}
```

## Consuming the TextSwitch control in XAML
 1. Add a reference to the custom control namespace in the XAML file:
 ```xaml
xmlns:controls="clr-namespace:CustomControlsSample.CustomControls"
```
 2. Initialize the bindable properties that define the states of the control:
```xaml
<controls:TextSwitch x:Name="textSwitch" Text="On" BackgroundColor="Black" UnselectedColor="Gray" SelectedColor="White" SelectionChanged="TextSwitch_SelectionChanged"/>
```
3. Attach a handler to the `SelectionChanged` event to handle the selection change in the code-behind file:
```csharp
private async void TextSwitch_SelectionChanged(object sender, EventArgs e)
{
    string message = textSwitch.IsOn ? "TextSwitch is on" : "TextSwitch is off";
    await DisplayAlert("TextSwitch", message, "OK");
}
```
