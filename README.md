[![NuGet Stats](https://img.shields.io/nuget/v/XamlNameReferenceGenerator.svg)](https://www.nuget.org/packages/XamlNameReferenceGenerator) [![downloads](https://img.shields.io/nuget/dt/XamlNameReferenceGenerator)](https://www.nuget.org/packages/XamlNameReferenceGenerator) ![Build](https://github.com/avaloniaui/Avalonia.NameGenerator/workflows/Build/badge.svg) ![License](https://img.shields.io/github/license/avaloniaui/Avalonia.NameGenerator.svg) ![Size](https://img.shields.io/github/repo-size/avaloniaui/Avalonia.NameGenerator.svg)

### C# `SourceGenerator` for Typed Avalonia `x:Name` References 

This is a [C# `SourceGenerator`](https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/) built for generating strongly-typed references to controls with `x:Name` (or just `Name`) attributes declared in XAML (or, in `.axaml`). The source generator will look for the `xaml` (or `axaml`) file with the same name as your partial C# class that is a subclass of `Avalonia.INamed` and parses the XAML markup, finds all XAML tags with `x:Name` attributes and generates the C# code.

### Getting Started

In order to get started, just install the NuGet package:

```
dotnet add package XamlNameReferenceGenerator
```

Or, if you are using [submodules](https://git-scm.com/docs/git-submodule), you can reference the generator as such:

```xml
<ItemGroup>
    <!-- Remember to ensure XAML files are included via <AdditionalFiles>,
         otherwise C# source generator won't see XAML files. -->
    <AdditionalFiles Include="**\*.xaml"/>
    <ProjectReference Include="..\Avalonia.NameGenerator\Avalonia.NameGenerator.csproj"
                      OutputItemType="Analyzer"
                      ReferenceOutputAssembly="false" />
</ItemGroup>
```

### Usage

After installing the NuGet package, declare your view class as `partial`. Typed C# references to Avalonia controls declared in XAML files will be generated for classes referenced by the `x:Class` directive in XAML files. For example, for the following XAML markup:

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="Sample.App.SignUpView">
    <TextBox x:Name="UserNameTextBox" x:FieldModifier="public" />
</Window>
```

A new C# partial class named `SignUpView` with a single `public` property named `UserNameTextBox` of type `TextBox` will be generated in the `Sample.App` namespace. We won't see the generated file, but we'll be able to access the generated property as shown below:

```cs
using Avalonia.Controls;

namespace Sample.App
{
    public partial class SignUpView : Window
    {
        public SignUpView()
        {
            // This method is generated. Call it before accessing any
            // of the generated properties. The 'UserNameTextBox'
            // property is also generated.
            InitializeComponent();
            UserNameTextBox.Text = "Joseph";
        }
    }
}
```

<img src="https://hsto.org/getpro/habr/post_images/d9f/4aa/a1e/d9f4aaa1eb450f5dd2fca66631bc16a0.gif" />

### Why do I need this?

The typed `x:Name` references might be useful if you decide to use e.g. [ReactiveUI code-behind bindings](https://www.reactiveui.net/docs/handbook/data-binding/):

```cs
// UserNameValidation and PasswordValidation are auto generated.
public partial class SignUpView : ReactiveWindow<SignUpViewModel>
{
    public SignUpView()
    {
        InitializeComponent();
        this.WhenActivated(disposables =>
        {
            this.BindValidation(ViewModel, x => x.UserName, x => x.UserNameValidation.Text)
                .DisposeWith(disposables);
            this.BindValidation(ViewModel, x => x.Password, x => x.PasswordValidation.Text)
                .DisposeWith(disposables);
        });
    }
}
```

### Advanced Usage

The `x:Name` generator can be configured via MsBuild properties that you can put into your C# project file (`.csproj`). Using such options, you can configure the generator behavior, the default field modifier, namespace and path filters. The generator supports the following options:

- `AvaloniaNameGeneratorBehavior`  
    Possible values: `OnlyProperties`, `InitializeComponent`  
    Default value: `InitializeComponent`  
    Determines if the generator should generate get-only properties, or the `InitializeComponent` method.

- `AvaloniaNameGeneratorDefaultFieldModifier`  
    Possible values: `internal`, `public`, `private`, `protected`  
    Default value: `internal`  
    The default field modifier that should be used when there is no `x:FieldModifier` directive specified.

- `AvaloniaNameGeneratorFilterByPath`  
    Posssible format: `glob_pattern`, `glob_pattern;glob_pattern`  
    Default value: `*`  
    The generator will process only XAML files with paths matching the specified glob pattern(s).  
    Example: `*/Views/*View.xaml`, `*View.axaml;*Control.axaml`

- `AvaloniaNameGeneratorFilterByNamespace`  
    Posssible format: `glob_pattern`, `glob_pattern;glob_pattern`  
    Default value: `*`  
    The generator will process only XAML files with base classes' namespaces matching the specified glob pattern(s).  
    Example: `MyApp.Presentation.*`, `MyApp.Presentation.Views;MyApp.Presentation.Controls`

The default values are given by:

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <AvaloniaNameGeneratorBehavior>InitializeComponent</AvaloniaNameGeneratorBehavior>
        <AvaloniaNameGeneratorDefaultFieldModifier>internal</AvaloniaNameGeneratorDefaultFieldModifier>
        <AvaloniaNameGeneratorFilterByPath>*</AvaloniaNameGeneratorFilterByPath>
        <AvaloniaNameGeneratorFilterByNamespace>*</AvaloniaNameGeneratorFilterByNamespace>
    </PropertyGroup>
    <!-- ... -->
</Project>
```

![](https://user-images.githubusercontent.com/6759207/107812261-7ddfea00-6d80-11eb-9c7e-67bf95d0f0d4.gif)

### What do the generated sources look like?

For [`SignUpView`](https://github.com/avaloniaui/Avalonia.NameGenerator/blob/main/src/Avalonia.NameGenerator.Sandbox/Views/SignUpView.xaml), we get the following generated output when the source generator is in the `InitializeComponent` mode:

```cs
// <auto-generated />

using Avalonia.Controls;
using Avalonia.Markup.Xaml;

namespace Sample.App
{
    partial class SampleView
    {
        internal global::Avalonia.NameGenerator.Sandbox.Controls.CustomTextBox UserNameTextBox;
        public global::Avalonia.Controls.TextBlock UserNameValidation;
        private global::Avalonia.Controls.TextBox PasswordTextBox;
        internal global::Avalonia.Controls.TextBlock PasswordValidation;
        internal global::Avalonia.Controls.ListBox AwesomeListView;
        internal global::Avalonia.Controls.TextBox ConfirmPasswordTextBox;
        internal global::Avalonia.Controls.TextBlock ConfirmPasswordValidation;
        internal global::Avalonia.Controls.Button SignUpButton;
        internal global::Avalonia.Controls.TextBlock CompoundValidation;

        public void InitializeComponent(bool loadXaml = true, bool attachDevTools = true)
        {
            if (loadXaml)
            {
                AvaloniaXamlLoader.Load(this);
            }

// This will be added only if you install Avalonia.Diagnostics.
#if DEBUG
            if (attachDevTools)
            {
                this.AttachDevTools();
            } 
#endif

            UserNameTextBox = this.FindControl<global::Avalonia.NameGenerator.Sandbox.Controls.CustomTextBox>("UserNameTextBox");
            UserNameValidation = this.FindControl<global::Avalonia.Controls.TextBlock>("UserNameValidation");
            PasswordTextBox = this.FindControl<global::Avalonia.Controls.TextBox>("PasswordTextBox");
            PasswordValidation = this.FindControl<global::Avalonia.Controls.TextBlock>("PasswordValidation");
            AwesomeListView = this.FindControl<global::Avalonia.Controls.ListBox>("AwesomeListView");
            ConfirmPasswordTextBox = this.FindControl<global::Avalonia.Controls.TextBox>("ConfirmPasswordTextBox");
            ConfirmPasswordValidation = this.FindControl<global::Avalonia.Controls.TextBlock>("ConfirmPasswordValidation");
            SignUpButton = this.FindControl<global::Avalonia.Controls.Button>("SignUpButton");
            CompoundValidation = this.FindControl<global::Avalonia.Controls.TextBlock>("CompoundValidation");
        }
    }
}
```

If you enable the `OnlyProperties` source generator mode, you get:

```cs
// <auto-generated />

using Avalonia.Controls;

namespace Avalonia.NameGenerator.Sandbox.Views
{
    partial class SignUpView
    {
        internal global::Avalonia.NameGenerator.Sandbox.Controls.CustomTextBox UserNameTextBox => this.FindControl<global::Avalonia.NameGenerator.Sandbox.Controls.CustomTextBox>("UserNameTextBox");
        public global::Avalonia.Controls.TextBlock UserNameValidation => this.FindControl<global::Avalonia.Controls.TextBlock>("UserNameValidation");
        private global::Avalonia.Controls.TextBox PasswordTextBox => this.FindControl<global::Avalonia.Controls.TextBox>("PasswordTextBox");
        internal global::Avalonia.Controls.TextBlock PasswordValidation => this.FindControl<global::Avalonia.Controls.TextBlock>("PasswordValidation");
        internal global::Avalonia.Controls.TextBox ConfirmPasswordTextBox => this.FindControl<global::Avalonia.Controls.TextBox>("ConfirmPasswordTextBox");
        internal global::Avalonia.Controls.TextBlock ConfirmPasswordValidation => this.FindControl<global::Avalonia.Controls.TextBlock>("ConfirmPasswordValidation");
        internal global::Avalonia.Controls.Button SignUpButton => this.FindControl<global::Avalonia.Controls.Button>("SignUpButton");
        internal global::Avalonia.Controls.TextBlock CompoundValidation => this.FindControl<global::Avalonia.Controls.TextBlock>("CompoundValidation");
    }
}
```
