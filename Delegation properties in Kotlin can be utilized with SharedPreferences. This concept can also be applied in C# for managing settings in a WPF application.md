# Delegation properties in Kotlin can be utilized with SharedPreferences. This concept can also be applied in C# for managing settings in a WPF application.


<br>
<br>

# in Kotlin 

<br>


`Declare`
---
```kt
class SharedPreferencesDelegate(
    private val context: Context,
    private val defaultValue: String = ""
) : ReadWriteProperty<Any?, String> {

    private val sharedPreferences by lazy {
        context.getSharedPreferences("app_sharedPreferences", Context.MODE_PRIVATE)
    }

    override fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return sharedPreferences.getString(property.name, defaultValue) ?: defaultValue
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        sharedPreferences.edit().putString(property.name, value).apply()
    }
}

fun Context.sharedPreferences() =
    SharedPreferencesDelegate(this)

fun Context.sharedPreferences(defaultValue: String) =
    SharedPreferencesDelegate(this, defaultValue)
```


<br>

`Usage`
---
```kt
class MainActivity : AppCompatActivity() {

    private var theme by sharedPreferences("light")

    override fun onCreate(savedInstanceState: Bundle?) {
        WindowCompat.setDecorFitsSystemWindows(window, false)
        super.onCreate(savedInstanceState)
        initializeUi()

        // get value from sharedPreferences
        Log.d("TAG", "current theme is $theme")

        // set value directly to sharedPreferences
        theme = "dark"
    }
}
```

<br>
<br>
<br>

# Use same concept in C# to work with settings in wpf app


`Declare`
---
```C#
using System;
using System.Configuration;
using System.Runtime.CompilerServices;

public class SettingsDelegate<T>
{
    private readonly T _defaultValue;

    public SettingsDelegate(T defaultValue = default!)
    {
        _defaultValue = defaultValue;
    }

    public T GetValue([CallerMemberName] string key = null!)
    {
        try
        {
            if (Properties.Settings.Default[key] is T value)
                return value;
            return _defaultValue;
        }
        catch
        {
            return _defaultValue;
        }
    }

    public void SetValue(T value, [CallerMemberName] string key = null!)
    {
        try
        {
            Properties.Settings.Default[key] = value;
            Properties.Settings.Default.Save();
        }
        catch (Exception ex)
        {
            throw new InvalidOperationException($"Error saving setting '{key}'", ex);
        }
    }
}
```

<br>

`Usage`
---
```C#
public class UserSettings
{
    private static readonly SettingsDelegate<string> _stringSetting = new("Guest");
    private static readonly SettingsDelegate<string> _themeSetting = new("Light");

    public string Username
    {
        get => _stringSetting.GetValue();
        set => _stringSetting.SetValue(value);
    }

    public string Theme
    {
        get => _themeSetting.GetValue();
        set => _themeSetting.SetValue(value);
    }
}

UserSettings userSettings = new();
userSettings.Username = "JohnDoe";
Console.WriteLine(userSettings.Username); // Output: JohnDoe
```
