# WooCommerce Integer Quantities Only

This code snippet provides a solution to **enforce integer (whole number) quantities for all products in your WooCommerce store**. By default, WooCommerce allows decimal quantities if a product's stock management or configuration permits. This modification sets the quantity input step to 1, ensuring customers can only select whole numbers, and also rounds down any accidental decimal quantities that might get added to the cart (e.g., via direct URL manipulation or browser quirks).

## Why enforce integer quantities?

* **Prevent Errors:** Avoids issues with products that are naturally sold in whole units (e.g., 1 shirt, 2 books, 5 kilograms if you only sell whole KGs).
* **Simplify User Input:** Makes the quantity selection clearer and less prone to confusion for customers.
* **Data Integrity:** Ensures that all quantities stored in your cart and orders are whole numbers, simplifying inventory management and reporting.
* **Consistent Experience:** Provides a uniform quantity selection method across your entire product catalog.

## Features

* **Sets Quantity Step to 1:** Modifies the quantity input field (`<input type="number" step="...">`) to only allow whole numbers (e.g., 1, 2, 3) for all products.
* **Enforces Minimum Quantity:** Sets the minimum purchasable quantity to 1.
* **Applies Maximum Quantity:** Uses the product's defined maximum purchase quantity if available.
* **Corrects Decimal Quantities in Cart:** Automatically rounds any decimal quantities in the cart to the nearest whole number before totals are calculated, preventing errors during checkout.

## Installation

There are two primary methods to implement this code:

### Method 1: As a Standalone Plugin (Recommended)

Creating a small, dedicated plugin is the most robust way to add this functionality. It ensures your modification remains active regardless of theme changes.

1.  Create a new folder named `wc-no-decimal-qty` inside your WordPress site's `wp-content/plugins/` directory.
2.  Inside this new folder, create a file named `wc-no-decimal-qty.php`.
3.  Copy and paste the following code into `wc-no-decimal-qty.php`:

    ```php
    <?php
    /**
     * Plugin Name: WooCommerce Integer Quantities Only
     * Description: Forces all WooCommerce product quantities to be whole numbers (integers) and corrects any accidental decimal quantities in the cart.
     * Version: 1.0
     * Author: Your Name (Optional)
     * License: GPL-2.0-or-later
     */

    if ( ! defined( 'ABSPATH' ) ) {
        exit; // Exit if accessed directly.
    }

    /**
     * Disables decimal quantities for WooCommerce product input fields.
     * Sets the step to 1 and ensures min/max values are respected.
     *
     * @param array    $args    Arguments for the quantity input.
     * @param WC_Product $product The product object.
     * @return array Modified arguments.
     */
    add_filter('woocommerce_quantity_input_args', 'disable_decimal_quantities', 10, 2);
    function disable_decimal_quantities($args, $product) {
        $args['step'] = 1; // Set step to one (integer only)
        $args['min_value'] = 1; // Minimum value
        $args['max_value'] = $product->get_max_purchase_quantity(); // Maximum value from product settings
        return $args;
    }

    /**
     * Checks and corrects any decimal quantities in the cart to be integers.
     * This acts as a fallback for direct URL manipulation or browser quirks.
     *
     * @param WC_Cart $cart The WooCommerce cart object.
     */
    add_action('woocommerce_before_calculate_totals', 'check_quantity_is_integer');
    function check_quantity_is_integer($cart) {
        // This condition ensures the code runs only once during the cart total calculation process.
        if ( is_admin() && ! defined( 'DOING_AJAX' ) ) {
            return;
        }

        foreach ($cart->get_cart() as $cart_item_key => $values) {
            $quantity = $values['quantity'];
            // If quantity is not a whole number (e.g., 2.5)
            if (floor($quantity) != $quantity) {
                // Round to the nearest whole number. You could also use ceil() or floor() based on preference.
                // Rounding ensures that 2.5 goes to 3, 2.4 goes to 2.
                $cart->set_quantity($cart_item_key, round($quantity));
            }
        }
    }
    ?>
    ```

4.  Go to your WordPress admin dashboard, navigate to **"Plugins"**, and **activate** the "WooCommerce Integer Quantities Only" plugin.

### Method 2: Adding to your Theme's functions.php File

You can add this code directly to your active theme's `functions.php` file. **Before doing so, it's highly recommended to back up your `functions.php` file.**

1.  Navigate to `wp-content/themes/YourThemeName/` (replace `YourThemeName` with the actual name of your active theme).
2.  Open the `functions.php` file.
3.  Add the following code to the end of the file (before the closing `?>` tag, if one exists):

    ```php
    /**
     * Disables decimal quantities for WooCommerce product input fields.
     * Sets the step to 1 and ensures min/max values are respected.
     *
     * @param array    $args    Arguments for the quantity input.
     * @param WC_Product $product The product object.
     * @return array Modified arguments.
     */
    add_filter('woocommerce_quantity_input_args', 'disable_decimal_quantities', 10, 2);
    function disable_decimal_quantities($args, $product) {
        $args['step'] = 1; // Set step to one (integer only)
        $args['min_value'] = 1; // Minimum value
        $args['max_value'] = $product->get_max_purchase_quantity(); // Maximum value from product settings
        return $args;
    }

    /**
     * Checks and corrects any decimal quantities in the cart to be integers.
     * This acts as a fallback for direct URL manipulation or browser quirks.
     *
     * @param WC_Cart $cart The WooCommerce cart object.
     */
    add_action('woocommerce_before_calculate_totals', 'check_quantity_is_integer');
    function check_quantity_is_integer($cart) {
        // This condition ensures the code runs only once during the cart total calculation process.
        if ( is_admin() && ! defined( 'DOING_AJAX' ) ) {
            return;
        }
        
        foreach ($cart->get_cart() as $cart_item_key => $values) {
            $quantity = $values['quantity'];
            // If quantity is not a whole number (e.g., 2.5)
            if (floor($quantity) != $quantity) {
                // Round to the nearest whole number. You could also use ceil() or floor() based on preference.
                // Rounding ensures that 2.5 goes to 3, 2.4 goes to 2.
                $cart->set_quantity($cart_item_key, round($quantity));
            }
        }
    }
    ```

## Customization

* **Rounding Behavior:** The `check_quantity_is_integer` function uses `round()` to round decimal quantities to the nearest whole number. If you prefer to always round down, you can change `round($quantity)` to `floor($quantity)`. If you prefer to always round up, change it to `ceil($quantity)`.
* **Minimum Value:** You can change `$args['min_value'] = 1;` to `0` if you want to allow a quantity of zero (though for most physical products, 1 is the practical minimum).

## Important Considerations

* **Product Type Compatibility:** This code is primarily for simple and variable products where quantities are typically integers. For products that genuinely need decimal quantities (e.g., selling by weight like 1.5 kg of flour), this code is not suitable and should not be used.
* **Existing Products:** This code affects new quantity inputs. For existing cart items or orders that might have had decimal quantities before this code was implemented, the `check_quantity_is_integer` function will correct them when the cart is next updated or totals are recalculated.
* **User Feedback:** This code doesn't explicitly display a message to the user if a decimal quantity is automatically rounded. The change happens silently. If you need explicit feedback, additional JavaScript or WooCommerce notices would be required.

## Contributing

Contributions are welcome! If you have suggestions or improvements for this code, feel free to open a "Pull Request" or report an "Issue."

## License

This project is licensed under the GPL-2.0-or-later License.

---

# فقط تعداد صحیح در ووکامرس

این قطعه کد راه حلی برای **اجرای اجباری تعداد صحیح (اعداد کامل) برای همه محصولات در فروشگاه ووکامرس شما** ارائه می‌دهد. به طور پیش‌فرض، ووکامرس اجازه می‌دهد در صورت اجازه مدیریت موجودی یا پیکربندی محصول، تعداد اعشاری نیز وارد شود. این تغییر گام ورودی تعداد را به ۱ تنظیم می‌کند و اطمینان می‌دهد که مشتریان فقط می‌توانند اعداد کامل را انتخاب کنند، همچنین هر تعداد اعشاری تصادفی را که ممکن است به سبد خرید اضافه شود (مثلاً از طریق دستکاری مستقیم URL یا اشکالات مرورگر)، گرد می‌کند.

## چرا تعداد صحیح را اجباری کنیم؟

* **جلوگیری از خطا:** از بروز مشکلات با محصولاتی که به طور طبیعی به صورت واحد کامل فروخته می‌شوند (مانند ۱ پیراهن، ۲ کتاب، ۵ کیلوگرم اگر فقط کیلوگرم کامل می‌فروشید) جلوگیری می‌کند.
* **ساده‌سازی ورودی کاربر:** انتخاب تعداد را برای مشتریان واضح‌تر و کمتر مستعد سردرگمی می‌کند.
* **یکپارچگی داده:** تضمین می‌کند که تمام مقادیر ذخیره شده در سبد خرید و سفارشات شما اعداد کامل هستند، که مدیریت موجودی و گزارش‌گیری را ساده می‌کند.
* **تجربه ثابت:** یک روش انتخاب تعداد یکنواخت را در کل کاتالوگ محصولات شما فراهم می‌کند.

## قابلیت‌ها

* **تنظیم گام تعداد به ۱:** فیلد ورودی تعداد (`<input type="number" step="...">`) را طوری تغییر می‌دهد که فقط اعداد کامل (مانند ۱، ۲، ۳) برای همه محصولات مجاز باشد.
* **اعمال حداقل مقدار:** حداقل مقدار قابل خرید را به ۱ تنظیم می‌کند.
* **اعمال حداکثر مقدار:** از حداکثر مقدار خرید تعریف شده محصول، در صورت وجود، استفاده می‌کند.
* **تصحیح تعداد اعشاری در سبد خرید:** به طور خودکار هر تعداد اعشاری در سبد خرید را قبل از محاسبه مجموع، به نزدیک‌ترین عدد کامل گرد می‌کند و از بروز خطا در هنگام تسویه‌حساب جلوگیری می‌کند.

## نصب

برای پیاده‌سازی این کد، دو روش اصلی وجود دارد:

### روش ۱: به عنوان یک افزونه مستقل (توصیه شده)

ایجاد یک افزونه کوچک و اختصاصی، قوی‌ترین راه برای افزودن این قابلیت است. این تضمین می‌کند که تغییر شما صرف‌نظر از تغییر قالب، فعال باقی بماند.

1.  یک پوشه جدید با نام `wc-no-decimal-qty` در مسیر `wp-content/plugins/` سایت وردپرسی خود ایجاد کنید.
2.  در داخل این پوشه جدید، یک فایل با نام `wc-no-decimal-qty.php` ایجاد کنید.
3.  کد زیر را در `wc-no-decimal-qty.php` کپی و جایگذاری کنید:

    ```php
    <?php
    /**
     * Plugin Name: WooCommerce Integer Quantities Only
     * Description: Forces all WooCommerce product quantities to be whole numbers (integers) and corrects any accidental decimal quantities in the cart.
     * Version: 1.0
     * Author: Your Name (Optional)
     * License: GPL-2.0-or-later
     */

    if ( ! defined( 'ABSPATH' ) ) {
        exit; // Exit if accessed directly.
    }

    /**
     * Disables decimal quantities for WooCommerce product input fields.
     * Sets the step to 1 and ensures min/max values are respected.
     *
     * @param array    $args    Arguments for the quantity input.
     * @param WC_Product $product The product object.
     * @return array Modified arguments.
     */
    add_filter('woocommerce_quantity_input_args', 'disable_decimal_quantities', 10, 2);
    function disable_decimal_quantities($args, $product) {
        $args['step'] = 1; // تعیین گام به یک (فقط عدد صحیح)
        $args['min_value'] = 1; // مقدار حداقل
        $args['max_value'] = $product->get_max_purchase_quantity(); // مقدار حداکثر
        return $args;
    }

    /**
     * Checks and corrects any decimal quantities in the cart to be integers.
     * This acts as a fallback for direct URL manipulation or browser quirks.
     *
     * @param WC_Cart $cart The WooCommerce cart object.
     */
    add_action('woocommerce_before_calculate_totals', 'check_quantity_is_integer');
    function check_quantity_is_integer($cart) {
        // این شرط اطمینان می‌دهد که کد فقط یک بار در طول فرآیند محاسبه مجموع سبد خرید اجرا می‌شود.
        if ( is_admin() && ! defined( 'DOING_AJAX' ) ) {
            return;
        }

        foreach ($cart->get_cart() as $cart_item_key => $values) {
            $quantity = $values['quantity'];
            // اگر تعداد یک عدد کامل نیست (مثلاً 2.5)
            if (floor($quantity) != $quantity) {
                // گرد کردن به نزدیک‌ترین عدد کامل. می‌توانید از ceil() یا floor() نیز بسته به ترجیح استفاده کنید.
                // گرد کردن تضمین می‌کند که 2.5 به 3 و 2.4 به 2 تبدیل می‌شود.
                $cart->set_quantity($cart_item_key, round($quantity));
            }
        }
    }
    ?>
    ```

4.  وارد پنل مدیریت وردپرس خود شوید، به بخش **"افزونه‌ها"** بروید و افزونه **"فقط تعداد صحیح در ووکامرس"** را **فعال کنید**.

### روش ۲: اضافه کردن به فایل functions.php قالب شما

می‌توانید این کد را مستقیماً به فایل `functions.php` قالب فعال خود اضافه کنید. **پیشنهاد اکید می‌شود قبل از انجام این کار، از فایل `functions.php` خود یک پشتیبان (backup) تهیه کنید.**

1.  به مسیر `wp-content/themes/YourThemeName/` بروید (به جای `YourThemeName` نام واقعی قالب فعال خود را قرار دهید).
2.  فایل `functions.php` را باز کنید.
3.  کد زیر را به انتهای فایل (قبل از تگ بستن `?>`، در صورت وجود) اضافه کنید:

    ```php
    /**
     * Disables decimal quantities for WooCommerce product input fields.
     * Sets the step to 1 and ensures min/max values are respected.
     *
     * @param array    $args    Arguments for the quantity input.
     * @param WC_Product $product The product object.
     * @return array Modified arguments.
     */
    add_filter('woocommerce_quantity_input_args', 'disable_decimal_quantities', 10, 2);
    function disable_decimal_quantities($args, $product) {
        $args['step'] = 1; // تعیین گام به یک (فقط عدد صحیح)
        $args['min_value'] = 1; // مقدار حداقل
        $args['max_value'] = $product->get_max_purchase_quantity(); // مقدار حداکثر
        return $args;
    }

    /**
     * Checks and corrects any decimal quantities in the cart to be integers.
     * This acts as a fallback for direct URL manipulation or browser quirks.
     *
     * @param WC_Cart $cart The WooCommerce cart object.
     */
    add_action('woocommerce_before_calculate_totals', 'check_quantity_is_integer');
    function check_quantity_is_integer($cart) {
        // این شرط اطمینان می‌دهد که کد فقط یک بار در طول فرآیند محاسبه مجموع سبد خرید اجرا می‌شود.
        if ( is_admin() && ! defined( 'DOING_AJAX' ) ) {
            return;
        }

        foreach ($cart->get_cart() as $cart_item_key => $values) {
            $quantity = $values['quantity'];
            // اگر تعداد یک عدد کامل نیست (مثلاً 2.5)
            if (floor($quantity) != $quantity) {
                // گرد کردن به نزدیک‌ترین عدد کامل. می‌توانید از ceil() یا floor() نیز بسته به ترجیح استفاده کنید.
                // گرد کردن تضمین می‌کند که 2.5 به 3 و 2.4 به 2 تبدیل می‌شود.
                $cart->set_quantity($cart_item_key, round($quantity));
            }
        }
    }
    ```

## سفارشی‌سازی

* **رفتار گرد کردن:** تابع `check_quantity_is_integer` از `round()` برای گرد کردن تعداد اعشاری به نزدیک‌ترین عدد کامل استفاده می‌کند. اگر ترجیح می‌دهید همیشه به پایین گرد شود، می‌توانید `round($quantity)` را به `floor($quantity)` تغییر دهید. اگر ترجیح می‌دهید همیشه به بالا گرد شود، آن را به `ceil($quantity)` تغییر دهید.
* **حداقل مقدار:** می‌توانید `$args['min_value'] = 1;` را به `0` تغییر دهید اگر می‌خواهید مقدار صفر نیز مجاز باشد (اگرچه برای بیشتر محصولات فیزیکی، ۱ حداقل مقدار عملی است).

## ملاحظات مهم

* **سازگاری با نوع محصول:** این کد در درجه اول برای محصولات ساده و متغیر است که در آن‌ها تعداد به طور معمول اعداد صحیح هستند. برای محصولاتی که واقعاً نیاز به تعداد اعشاری دارند (مثلاً فروش بر اساس وزن مانند ۱.۵ کیلوگرم آرد)، این کد مناسب نیست و نباید استفاده شود.
* **محصولات موجود:** این کد بر ورودی‌های جدید تعداد تأثیر می‌گذارد. برای اقلام سبد خرید یا سفارشات موجود که ممکن است قبل از اجرای این کد دارای تعداد اعشاری بوده‌اند، تابع `check_quantity_is_integer` آن‌ها را هنگام به‌روزرسانی بعدی سبد خرید یا محاسبه مجدد مجموع، تصحیح خواهد کرد.
* **بازخورد کاربر:** این کد به صراحت پیامی را به کاربر نشان نمی‌دهد اگر یک تعداد اعشاری به طور خودکار گرد شود. تغییر به صورت بی‌صدا اتفاق می‌افتد. اگر به بازخورد صریح نیاز دارید، نیاز به جاوااسکریپت یا اعلان‌های ووکامرس اضافی خواهید داشت.

## مشارکت (Contributing)

مشارکت شما خوشایند است! اگر پیشنهاد یا بهبودهایی برای این کد دارید، می‌توانید یک "Pull Request" ایجاد کنید یا "Issue" جدیدی را گزارش دهید.

## مجوز (License)

این پروژه تحت مجوز GPL-2.0-or-later منتشر شده است.
