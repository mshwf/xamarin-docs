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

The process of building UI elements requires, at some point, some customizations to give the unique feel and look to the application and to extend the functionality of existing controls. Whether the customization is just overriding the default TextColor of the Entry or creating a brand new control with new look and behavior, building custom controls can provide the simple solution to achieve that, without the need of custom renderers.

The process for creating custom controls is as follows:

1. Create a subclass of the view you want to extend or modify.
2. Alter the functionality of the subclass by overriding the default value of the base class’s bindable properties and/or create new bindable properties that will interact with user actions.
3. Process inputs through the `propertyChanged` delegate of the newly added bindable properties.

## Create a custom Toggle bar

The toggle bar control is used to show some options that the user can choose from, for example a filtering mechanism or a light-weight tabbed control..etc, see the screenshot below (should look the same on iOS): 
[screenshot (on Android)]

The behavior of the control is as follows:

1. Every button has a selected state and unselected state determined by the `IsSelected` property.
2. The states are visually distinguished through the `SelectedColor` and `UnselectedColor` bindable properties.
3. The selected Items can be obtained through the bindable property `SelectedItems`.
4. The control supports multi-selection (that’s why it’s `SelectedItems` not `SelectedItem`) by setting the `IsMultiSelect` property to `true` (defaults to `false`).

Bindable properties is the foundation of custom controls (For more information about Xamarin.Forms bindable properties, see [Xamarin.Forms Bindable Properties
](~/xamarin-forms/xaml/bindable-properties.md)

Before creating the Toggles bar control, we need first to create the single Toggle button control, the steps are as follows:
1. Create a subclass from StackLayout, name it ToggleButton, it holds two children: `Label` and `BoxView`, The following diagram illustrates the control outline:

