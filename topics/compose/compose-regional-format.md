# Manage regional formats

Text fields and related components, such as date and time pickers, automatically apply locale-specific formatting patterns.
These include the formatting of numbers, dates, times, and currencies. 
The patterns are determined by the system's locale or a custom locale specified within your application, 
ensuring consistent data display across platforms.

## Locale formatting patterns

Formatting in Compose Multiplatform respects platform-specific locale conventions for:

 * Numeric separators. Thousands can get separated by a comma or a period: `1,234.56` in `en-US` and `1.234,56` in `de-DE`.
 * Date/Time patterns. The date of December 1 looks like `12/1/20` in `en-US`, 
   while the same date in `fr-CA` looks like `2020-12-01`.
 * Currency symbols and placement. The local currency symbol can be placed either before or after the digits: 
   `$5,001` in `en-US` and `5.001,00 €` in `fr-FR`.

These regional formats work out of the box for both system locales and custom locale overrides defined in your app.

Formatting is currently implemented using the `kotlinx-datetime` library on iOS and the JDK API on Android and desktop.
Here is an example of managing localized date and currency values in the iOS source set:

```kotlin
import androidx.compose.foundation.layout.Column
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import kotlinx.datetime.Clock
import kotlinx.datetime.TimeZone
import kotlinx.datetime.todayIn
import platform.Foundation.NSNumber
import platform.Foundation.NSNumberFormatter
import platform.Foundation.NSNumberFormatterCurrencyStyle
import platform.Foundation.NSLocale
import platform.Foundation.currentLocale

@Composable
fun RegionalFormatExample() {
    val currentLocale = NSLocale.current
    val languageCode = currentLocale.languageCode 
    val countryCode = currentLocale.countryCode 

    // Sample value to format as currency
    val amount = 1234.56

    // Uses NSNumberFormatter for currency formatting on iOS
    val currencyFormatter = NSNumberFormatter().apply { 
        numberStyle = NSNumberFormatterCurrencyStyle
        locale = currentLocale
    }
    val formattedCurrency = currencyFormatter.stringFromNumber(NSNumber(amount)) 

    // Gets current date using `kotlinx-datetime`
    val today = Clock.System.todayIn(TimeZone.currentSystemDefault())
    val formattedDate = today.toString()

    Column {
        Text("Current language: $languageCode")
        Text("Current country: $countryCode")
        Text("Amount: $amount")
        Text("Formatted currency: $formattedCurrency")
        Text("Today's date: $formattedDate")
    }
}
```

## Ensure consistent formatting

Although there is no common API for a unified multiplatform solution, the formatting behavior is still consistent in most cases.
To ensure the formatting is correct for all supported regions:

* Test edge cases such as large numbers, negative values, or zeroes.
* Verify formatting for all supported locales on all target platforms.

## What's next

Learn how to manage the application's [resource environment](compose-resource-environment.md) like in-app theme and language.