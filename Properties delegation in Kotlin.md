### Property Delegation في Kotlin: تبسيط إدارة الإعدادات باستخدام SharedPreferences وتطبيق المفهوم في C#

إدارة إعدادات التطبيقات بشكل فعال تُعد جزءًا أساسيًا من تطوير البرمجيات، سواء كنت تعمل على تطبيقات Android باستخدام Kotlin أو على تطبيقات WPF باستخدام C#. في هذا المقال، سنستكشف مفهوم **Property Delegation** في Kotlin وكيفية استخدامه مع SharedPreferences، ثم ننتقل إلى كيفية تحقيق مفهوم مشابه في C# باستخدام إعدادات WPF.

---

## ما هو Property Delegation في Kotlin؟

Property Delegation هو نمط تصميم يُستخدم في Kotlin لتمكين الكائنات من تفويض إدارة خصائصها إلى كائن آخر. يساعد هذا المفهوم في كتابة كود أكثر قابلية لإعادة الاستخدام والتنظيم، خاصة عند التعامل مع حالات شائعة مثل إدارة الإعدادات أو الكسل (laziness).

## تطبيق Property Delegation مع SharedPreferences

### تعريف SharedPreferencesDelegate:

في Kotlin، تُستخدم SharedPreferences لتخزين البيانات البسيطة مثل النصوص أو القيم الرقمية. يمكننا تحسين عملية الوصول إلى هذه البيانات باستخدام Property Delegation، كما في المثال التالي:

```kotlin
import android.content.Context
import kotlin.properties.ReadWriteProperty
import kotlin.reflect.KProperty

class SharedPreferencesDelegate(
    private val context: Context,
    private val defaultValue: String = ""
) : ReadWriteProperty<Any?, String> {

    private val sharedPreferences by lazy {
        context.getSharedPreferences("app_shared_preferences", Context.MODE_PRIVATE)
    }

    override fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return sharedPreferences.getString(property.name, defaultValue) ?: defaultValue
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        sharedPreferences.edit().putString(property.name, value).apply()
    }
}

fun Context.sharedPreferences(defaultValue: String = "") =
    SharedPreferencesDelegate(this, defaultValue)
```

### كيفية الاستخدام:

```kotlin
class UserSettings(context: Context) {
    var username: String by context.sharedPreferences("Guest")
    var theme: String by context.sharedPreferences("Light")
}

// الاستخدام في التطبيق
val userSettings = UserSettings(context)
userSettings.username = "JohnDoe"
println(userSettings.username) // Output: JohnDoe
```

---

## تطبيق مفهوم مماثل في C# مع WPF Settings

### ما هي إعدادات WPF؟

في C#، تُستخدم إعدادات WPF (“Application Settings”) لتخزين واسترجاع القيم البسيطة بطريقة مشابهة لـ SharedPreferences في Kotlin. يمكننا تبسيط إدارة هذه الإعدادات باستخدام مندوب Generics.

### تعريف SettingsDelegate:

```csharp
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

### كيفية الاستخدام:

```csharp
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

// الاستخدام في التطبيق
var userSettings = new UserSettings();
userSettings.Username = "JohnDoe";
Console.WriteLine(userSettings.Username); // Output: JohnDoe
```

---

## مقارنة بين Kotlin و C#

| **المفهوم**          | **Kotlin**                   | **C#**                      |
| -------------------- | ---------------------------- | --------------------------- |
| **تخزين البيانات**   | SharedPreferences            | Application Settings        |
| **مندوب الإعدادات**  | ReadWriteProperty            | SettingsDelegate            |
| **التطبيق**          | Context.sharedPreferences()  | Properties.Settings.Default |
| **مرونة الأنواع**    | نعم، باستخدام `String` كمثال | نعم، باستخدام Generics      |
| **التخزين التلقائي** | نعم باستخدام `apply()`       | نعم باستخدام `Save()`       |

---

## الخاتمة

يُعد استخدام Property Delegation أسلوبًا قويًا لتبسيط إدارة الإعدادات في التطبيقات. في Kotlin، يوفر هذا المفهوم مرونة كبيرة عند العمل مع SharedPreferences. وعلى الجانب الآخر، يمكن تحقيق نفس الفائدة في C# باستخدام مندوب Generics مع إعدادات WPF.

إذا كنت تعمل على تطبيق متعدد المنصات، فإن تبني هذه الأفكار يساعدك على الحفاظ على كود نظيف ومنظم، مما يسهم في تحسين تجربة التطوير بشكل عام.

