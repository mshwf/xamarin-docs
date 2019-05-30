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

## Create a custom Toggle bar

The toggle bar control is used to show some options that the user can choose from, for example a filtering mechanism or a light-weight tabbed control..etc, similar to a group of radio buttons, see the screenshot below (should look the same on iOS):
[screenshot (on Android)]

The behavior of the control is as follows:

1. Every button has a selected state and unselected state determined by the `IsSelected` property.
2. The states are visually distinguished through the `SelectedColor` and `UnselectedColor` bindable properties.
3. The selected items can be obtained through the bindable property `SelectedItems`.
4. The control supports multi-selection (that’s why it’s `SelectedItems` not `SelectedItem`) by setting the `IsMultiSelect` property to `true` (defaults to `false`).

Bindable properties is the foundation of custom controls (For more information about Xamarin.Forms bindable properties, see [Xamarin.Forms Bindable Properties
](~/xamarin-forms/xaml/bindable-properties.md))

Before creating the Toggles bar control, we need first to create the single Toggle button control, the steps are as follows:
1. Create a subclass from `StackLayout`, name it `ToggleButton`, it holds two children: `Label` and `BoxView`, the following diagram illustrates the control outline:
![](custom-controls-images/togglebutton-layout.png "Togle bar control outline")

When the label is tapped the selection state is mutated. The visual state is defined by the `TextColor` property of the Label and the `Color` property of the BoxView,

2. Create the bindable properties: `IsSelected`, `SelectedColor`, `UnselectedColor`, `Text` and `FontFamily`. This is the `SelectedColor` property along with the [`BindableProperty`](xref:Xamarin.Forms.BindableProperty) backing field:

```csharp
public static readonly BindableProperty SelectedColorProperty = BindableProperty.Create(nameof(SelectedColor), typeof(Color), typeof(ToggleButton),
defaultValue: default(Color), propertyChanged: CustomPropertyChanged);

 public Color SelectedColor
 {
     get { return (Color)GetValue(SelectedColorProperty); }
     set { SetValue(SelectedColorProperty, value); }
 }
 ```
 
> [!NOTE]
> A bindable property is a special type of property, where the property's value is tracked by the Xamarin.Forms property system.
> The process of creating a bindable property is as follows:
> 1. Create a [`BindableProperty`](xref:Xamarin.Forms.BindableProperty) instance with one of the [`BindableProperty.Create`](xref:Xamarin.Forms.BindableProperty.Create*) method overloads.
> 2. Define property accessors for the [`BindableProperty`](xref:Xamarin.Forms.BindableProperty) instance.
>   
> For more information about Xamarin.Forms bindable properties, see [Xamarin.Forms Bindable Properties
> ](~/xamarin-forms/xaml/bindable-properties.md)

3. Attach the `propertyChanged` delegate of the bindable properties to a new method- `CustomPropertyChanged`- that will process inputs from the user, like setting the label's `Text` and `TextColor` properties from the `ToggleButton`'s `Text` and `UnselectedColor` properties respectively, and add a `TapGestureRecognizer` to the Label’s `GestureRecognizers` collection that will mutate the selection state of the toggle button when the label is tapped.

```csharp
private static void CustomPropertyChanged(BindableObject bindable, object oldValue, object newValue)
{
    if (newValue == null) return;
    ((ToggleButton)bindable).Render();
}

private void Render()
{
    button = new Label
    {
        TextColor = UnselectedColor,
        Text = Text,
        BackgroundColor = BackgroundColor,
        FontFamily = FontFamily,
        HorizontalTextAlignment = TextAlignment.Center,
        VerticalTextAlignment = TextAlignment.Center,
        Margin = new Thickness(5)
    };

    box = new BoxView { HeightRequest = 2, Color = BackgroundColor };

    Children.Clear();
    Children.Add(button);
    Children.Add(box);
    button.GestureRecognizers.Add(new TapGestureRecognizer()
    {
        Command = new Command(() =>
        {
            IsSelected = !IsSelected;
            SelectionChanged?.Invoke(this, EventArgs.Empty);
        })
    });


}
```
Create `SelectionChanged` event that gets invoked when the label is tapped, and it will notify consumers of the ToggleButton when selection changes:

`public event EventHandler SelectionChanged;`

One more step is to call The `MutateSelect` method in the set accessor of the `IsSelected` property:
```csharp
public bool IsSelected
{
    get { return (bool)GetValue(IsSelectedProperty); }
    set
    {
        SetValue(IsSelectedProperty, value);
        MutateSelect();
    }
}
```

The `MutateSelect` method is where the selection state gets updated visually when the `IsSelected` is mutated:

```csharp
void MutateSelect()
{
    if (IsSelected)
    {
        button.TextColor = SelectedColor;
        box.Color = SelectedColor;
    }
    else
    {
        button.TextColor = UnselectedColor;
        box.Color = BackgroundColor;
    }
}
```

Now the control can be consumed in XAML:
```xaml
<controls:ToggleButton Text="On" BackgroundColor="Black" UnselectedColor="Gray" SelectedColor="White" SelectionChanged="ToggleButton_SelectionChanged"/>
```

In the next part you are going to create the ToggleBar control making use of the control you just created.

## Toggle bar control

