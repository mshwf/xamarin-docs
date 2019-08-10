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
2. The states are visually distinguished through the `OnColor` and `OffColor` bindable properties.
3. When the control is tapped, the toggle state is mutated and the text color is updated to the `OnColor` or `OffColor`.

Bindable properties is the foundation of custom controls (For more information about Xamarin.Forms bindable properties, see [Xamarin.Forms Bindable Properties
](~/xamarin-forms/xaml/bindable-properties.md))

The steps of creating the TextSwitch control are as follows:
1. Create a subclass from `StackLayout`, name it `TextSwitch`, it holds two children: `Label` and `BoxView`, the following diagram illustrates the control outline:
![](custom-controls-images/togglebutton-layout.png "Togle bar control outline")

When the label is tapped, the toggle state is mutated. The visual state is defined by the `TextColor` property of the Label and the `Color` property of the BoxView,

2. Create the bindable properties: `IsOn`, `OnColor`, `OffColor`, `Text`, `FontFamily` and `FontSize`. This is the `OnColor` property along with the [`BindableProperty`](xref:Xamarin.Forms.BindableProperty) backing field:

```csharp
public static readonly BindableProperty OnColorProperty = BindableProperty.Create(nameof(OnColor), typeof(Color), typeof(TextSwitch),
defaultValue: default(Color), propertyChanged: CustomPropertyChanged);

 public Color OnColor
 {
     get { return (Color)GetValue(OnColorProperty); }
     set { SetValue(OnColorProperty, value); }
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

3. Attach the `propertyChanged` delegate of the bindable properties to `CustomPropertyChanged` method, that will process inputs from the user, like setting the label's `Text` and `TextColor` properties from the `TextSwitch`'s `Text` and `OffColor` properties respectively, and add a `TapGestureRecognizer` to the Label’s `GestureRecognizers` collection that will mutate the toggle state of the `TextSwitch` when the label is tapped.

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
        TextColor = OffColor,
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
            Toggled?.Invoke(this, EventArgs.Empty);
        })
    });


}
```
The `CustomPropertyChanged` is called whenever the bindable property, which [`propertyChanged`](xref:Xamarin.Forms.BindableProperty.BindingPropertyChangedDelegate) delegate is attached to, changes, a change occurs when the property is set when the custom control is initialized. Typically, bindable properties' `propertyChanged` delegate are attached to the same method, when they're responsible for customizing the control's appearance, because we want to make sure all properties are updated once another property changes, for that reason, the `IsOn` bindable property isn't attached to a `propertyChanged` delegate beacuse no UI customization is required based on its initial value, but if the value of the `IsOn` property is set, for example, from data binding, then `propertyChanged` is required to initialize the control with the correct state based on the `IsOn` property value, in this tutorial, the control is always initialized in off state (`IsOn` value is `false`).

> [!NOTE]
> When the custom control is initialized, `propertyChanged` delegate is called in the same order as the properties initialization order in XAML (or code), so for properties attached to the same delegate, the last call to the delegate handler is where all properties have been set.

The `Render` method initializes the control properties, for example the `TextColor` property of the label gets the value of `OffColor` property of the custom control beacause the control is rendered in off state, similarly, the `BoxView`'s color is initialized with the color of the `BackgroundColor` of the `StackLayout` to hide it, it only gets highlited with `OnColor` color when the control is on. Setting the `WidthRequest` and `HeightRequest` for both the `Label` and `BoxView` ensures they scale with their parent's size.

4. Create `Toggled` event that gets invoked when the label is tapped, and it will notify consumers of the TextSwitch when toggle changes:

```csharp
public event EventHandler Toggled;
```

When the label is tapped we need to change the toggle state of the control, create `Toggle` method and call it in the set accessor of the `IsOn` property that gets mutated when the label is tapped:
```csharp
public bool IsOn
{
    get { return (bool)GetValue(IsOnProperty); }
    set
    {
        SetValue(IsOnProperty, value);
        Toggle();
    }
}
```

The `Toggle` method is where the toggle state gets updated visually when the `IsOn` is mutated:

```csharp
void Toggle()
{
    if (IsOn)
    {
        button.TextColor = OnColor;
        underLine.Color = OnColor;
    }
    else
    {
        button.TextColor = OffColor;
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
<controls:TextSwitch x:Name="textSwitch" Text="On" BackgroundColor="Black" OffColor="Gray" OnColor="White" Toggled="TextSwitch_Toggled"/>
```
3. Attach a handler to the `Toggled` event to handle the toggle change in the code-behind file:
```csharp
private async void TextSwitch_Toggled(object sender, EventArgs e)
{
    string message = textSwitch.IsOn ? "TextSwitch is on" : "TextSwitch is off";
    await DisplayAlert("TextSwitch", message, "OK");
}
```
