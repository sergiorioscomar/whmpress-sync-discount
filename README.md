
# ❓¿Cómo lograste que los precios del WHMCS se actualicen solos en WordPress y además mostrar descuentos personalizados?

En este post te muestro cómo trabajé con **WHMpress**, el plugin que conecta **WHMCS** con **WordPress**, para lograr dos cosas:

✅ Automatizar la sincronización diaria de precios desde WHMCS.  
✅ Personalizar los templates para aplicar **descuentos visuales** desde WordPress.

---

## 🔁 Sincronización automática con WHMCS vía Cron

WHMpress tiene una función de sincronización manual desde el panel de WordPress. Pero para automatizarla todos los días, podés crear un archivo `cron.php` y ejecutarlo con un cron job.

### 📄 cron.php

```php
<?php

require_once(__DIR__ . '/wp-load.php');

if (!function_exists("whmpress_cron_function")) {
    function whmpress_cron_function() {
        echo "Starting WHMPress cron job.<br>";
        echo "===========================<br>";
        echo whmp_fetch_data();
        echo "============================<br>";
        echo "WHMPress cron job completed.<br>";
    }
}

whmpress_cron_function();
```

### 📌 Cron Job

```bash
0 3 * * * php /var/www/html/sitio/cron.php >/dev/null 2>&1
```

Este comando se ejecuta todos los días a las 3 AM y sincroniza datos desde WHMCS a WordPress.

---

## 🧱 Agregar descuento personalizado en los templates de WHMpress

Al usar Elementor podés insertar widgets como `whmpress_pricing_table`. WHMpress permite seleccionar:

- Producto/servicio  
- Template visual  
- Moneda y ciclo de facturación  

Además, se pueden agregar nuevos **atributos personalizados**, como descuentos.

### 🔧 Ejemplo: Agregar descuento en el widget y shortcode

```php
extract( shortcode_atts( [
    'discount' => whmpress_get_option("discount"),
    // otros atributos
], $atts ) );
```

Luego, en el array `$vars` que se pasa al template:

```php
$vars = [
    "discount" => $discount,
    "amountwithdiscount" => floor($tmp2['amount'] * (1 - $discount / 100)),
];
```

Esto calcula automáticamente el precio con descuento.

---

## 🧩 Configurar el widget de Elementor (`pricing-table.php`)

Ubicación: `/wp-content/plugins/whmpress/widgets/pricing-table.php`

Agregar el control del descuento:

```php
$this->add_control(
    'discount',
    [
        'label' => 'Descuento',
        'type' => \Elementor\Controls_Manager::NUMBER,
        'default' => '0',
    ]
);
```

En el método `render`, obtenemos el valor:

```php
$discount = $settings['discount'];
```

Y lo pasamos al shortcode:

```php
echo do_shortcode('[whmpress_pricing_table 
    id="' . $id . '" 
    discount="' . $discount . '"
    // otros parámetros
]');
```

---

## 📄 Template `.tpl` de WHMpress con descuento

Ubicación: `/wp-content/plugins/whmpress/templates/whmpress_pricing_table/default`

Usamos Smarty para aplicar condicionales al render:

```smarty
{if isset($amountwithdiscount) && $amountwithdiscount != $amount}
    <span class="original-amount" style="text-decoration: line-through;">{$prefix}{$amount}</span>
    <span class="h-discount-tag">{$discount}% OFF</span><br>
    <div class="holder">
        <span class="currency">{$prefix}</span><span class="amount">{$amountwithdiscount|number_format:0:',':'.'}</span>
{/if}
```

---

## 💰 Moneda desde sesión o cookie (relacionado con otro post)

```php
if (empty($currency)) {
    if (isset($_SESSION["whmpress_currency"])) {
        $currency = $_SESSION["whmpress_currency"];
    }
}
```

Esto permite usar la moneda activa establecida por IP o por selección manual.

---

## ✅ Resultado

- Precios siempre actualizados automáticamente desde WHMCS  
- Descuentos visuales aplicables desde WordPress con Elementor  
- Templates totalmente personalizados  
- Moneda dinámica y coherente con el visitante  

---

🧠 Si querés ver cómo configuré la moneda según IP y mostré banderas, leé este post complementario 👉  
🔗 [¿Cómo integraste WHMCS con WordPress usando WHMpress y además lo personalizaste con código propio?](https://blog.sergiorios.com.ar/trabajo/%e2%9d%93como-integraste-whmcs-con-wordpress-usando-whmpress-y-ademas-lo-personalizaste-con-codigo-propio/)
