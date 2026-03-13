# Localization Guide

Winslop uses the standard WinUI 3 resource system (`.resw` files) for translations.
This guide explains how translations work and how you can contribute.

---

## For Translators

You don't need Visual Studio, C#, or any programming knowledge.
All you need is a text editor and a GitHub account.

### How translation files work

Every language has its own folder under `Strings/` with a `Resources.resw` file:

```
Strings/
  en-US/Resources.resw    <-- English (default)
  de-DE/Resources.resw    <-- German
```

The `.resw` file is simple XML. Each line has a **key** (don't touch) and a
**value** (translate this):

```xml
<data name="NavHome.Text"><value>Home</value></data>
```

The file is organized in clearly labeled sections so you always know
what part of the app you're translating:

```xml
<!-- XAML x:Uid: MainWindow — Navigation -->
<!-- XAML x:Uid: InstallPage -->
<!-- C# ResourceLoader: Donation dialog -->
```

### How to create a translation

1. **Copy** `Strings/en-US/Resources.resw` into a new folder, e.g. `Strings/fr-FR/`
2. **Translate** every `<value>...</value>` — do NOT change the `name` attributes
3. **Keep placeholders** like `{0}`, `{1}` — they get replaced with numbers/text at runtime
4. **Keep special characters** like `&lt;` (this is `<` in XML) and `&#x2764;` (heart emoji)
5. **Don't translate** brand names: Winslop, Copilot, Edge, Ko-fi, PayPal, CFEnhancer

**Example:**

```xml
<!-- English original -->
<data name="BtnAnalyzeText.Text"><value>Inspect system</value></data>
<data name="Tools_Loaded"><value>{0} extensions loaded.</value></data>

<!-- French translation -->
<data name="BtnAnalyzeText.Text"><value>Inspecter le système</value></data>
<data name="Tools_Loaded"><value>{0} extensions chargées.</value></data>
```

### How to submit your translation

1. Fork the repo: https://github.com/builtbybel/Winslop
2. Add your `Strings/<locale>/Resources.resw` file
3. Open a Pull Request with the title: **Add \<Language\> translation**

That's it! The developer will handle the project configuration, menu entry,
and compilation. You just provide the translated `.resw` file.

**Tips for a good translation:**
- Keep the same tone — Winslop is informal and direct
- The section comments in the file tell you which page each group belongs to
- If a value looks technical or unclear, check the English version in context
  by looking at the section header (e.g. "ToolsPage", "Donation dialog")

---

## For Developers

Technical details for integrating a new language into the build and UI.

### 1. Register the language

Add the locale to `SatelliteResourceLanguages` in `Winslop.csproj`:

```xml
<SatelliteResourceLanguages>en-US;de-DE;fr-FR</SatelliteResourceLanguages>
```

### 2. Add a menu entry

In `MainWindow.xaml`, add a new `MenuFlyoutItem` inside the language
switcher `MenuFlyoutSubItem`:

```xml
<MenuFlyoutItem x:Name="menuLangFrench" Text="Français" Click="MenuLangFrench_Click">
    <MenuFlyoutItem.Icon>
        <FontIcon x:Name="iconLangFrench" Glyph="&#xE73E;" Visibility="Collapsed" />
    </MenuFlyoutItem.Icon>
</MenuFlyoutItem>
```

### 3. Add the click handler

In `MainWindow.xaml.cs`:

```csharp
private async void MenuLangFrench_Click(object sender, RoutedEventArgs e)
{
    await Localizer.SwitchLanguageAsync("fr-FR", Content.XamlRoot);
    UpdateLanguageCheckmarks();
}
```

Update `UpdateLanguageCheckmarks()` to include the new icon:

```csharp
if (iconLangFrench != null)
    iconLangFrench.Visibility = lang == "fr-FR" ? Visibility.Visible : Visibility.Collapsed;
```

### 4. Build and test

Build the project. `MakePri` automatically compiles all `.resw` files
into `Winslop.pri`. Switch to the new language via **Settings > Language**
and restart the app.

### Key types reference

| Pattern | Used by | Example |
|---|---|---|
| `Name.Property` | XAML `x:Uid` (auto-applied at load) | `NavHome.Text`, `SearchBox.PlaceholderText` |
| `Name` | C# via `Localizer.Get()` / `GetFormat()` | `Analysis_Complete`, `Tools_Loaded` |

### File structure

| File | Purpose |
|---|---|
| `Strings/en-US/Resources.resw` | English strings (fallback) |
| `Strings/de-DE/Resources.resw` | German strings |
| `Helpers/Localizer.cs` | `Get()`, `GetFormat()`, `SwitchLanguageAsync()`, `CurrentLanguage` |
| `Helpers/SettingsHelper.cs` | Persists language choice to `Winslop.txt` |
| `App.xaml.cs` | Applies `PrimaryLanguageOverride` on startup |

---

## Questions?

Open an issue at https://github.com/builtbybel/Winslop/issues
