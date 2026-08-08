---
name: magento2-system-config-field-creator
description: >
  Implement system configuration fields in a Magento 2 project.
  Use this skill whenever you need to add or change a
  store configuration field, and whenever a module needs a new
  admin-configurable setting or default value.
---

# Goal

Add a configuration field to the Magento Admin panel
(`Admin > Stores > Configuration`) and expose 
its value to the rest of the
module through a dedicated system config manager, so that no other
class ever touches 
`\Magento\Framework\App\Config\ScopeConfigInterface` or a raw config path
string directly.

The examples below use `Vendor\Module` placeholders and a
`section/group/field` path.

Code snippets show only the parts relevant to each step. Other
required boilerplate (file docblock headers, constructors, additional
interface methods, etc.) is omitted and marked with `...`.

## Step 1: Declare the field in `system.xml`

**File:** `app/code/Vendor/Module/etc/adminhtml/system.xml`

Write **one attribute per line, vertically aligned**, instead of a
single long attribute list. Field declarations carry many attributes
and quickly exceed a readable line width; stacking them keeps every
attribute visible and makes diffs show exactly which attribute
changed.

```xml
<?xml version="1.0"?>
...
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <section id="section_id">
            <group id="group_id"
                   translate="label"
                   type="text"
                   sortOrder="100"
                   showInDefault="1"
                   showInWebsite="1"
                   showInStore="1">
                <label>System</label>
                <field id="field_active"
                       translate="label"
                       type="select"
                       sortOrder="100"
                       showInDefault="1"
                       showInWebsite="0"
                       showInStore="0"
                       canRestore="1">
                    <label>Field Enabled</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="field_ttl"
                       translate="label"
                       type="text"
                       sortOrder="200"
                       showInDefault="1"
                       showInWebsite="0"
                       showInStore="0"
                       canRestore="1">
                    <label>Field TTL (seconds)</label>
                    <comment>
                    <![CDATA[
                        <p class="note"><b>Note:</b> Explain here anything that is not obvious from the label.</p>
                    ]]>
                    </comment>
                </field>
            </group>
        </section>
    </system>
</config>
```

Guidelines:

- Use `sortOrder` in steps of `100` so fields can be inserted later
  without renumbering the whole group.
- Set `canRestore="1"` on every field that has a default in
  `config.xml`, so the Admin panel offers *Use system value*.
- Keep `showInDefault` / `showInWebsite` / `showInStore` consistent
  with the scope the value is actually read at. A field shown at
  store scope but read at default scope is a bug waiting to happen.
- Put non-obvious business context in a `<comment>` with `CDATA`,
  not in an XML comment — the admin user is the audience.
- Reuse an existing `<section>` and `<group>` when 
  the module extends a native
  area (`shipping`, `carriers`, `catalog`, ...).

## Step 2: Define default values in `config.xml`

**File:** `app/code/Vendor/Module/etc/config.xml`

Every field declared in `system.xml` should have a default here.
Without one, the value is `null` until somebody saves the form.

```xml
<?xml version="1.0"?>
...
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Store:etc/config.xsd">
    <default>
        <section_id>
            <group_id>
                <field_active>0</field_active>
                <field_ttl>604800</field_ttl>
            </group_id>
        </section_id>
    </default>
</config>
```

### Scoping defaults per website or store view

`<default>` is only one of the three scope nodes allowed by
`urn:magento:module:Magento_Store:etc/config.xsd`. Use `<websites>`
or `<stores>` when a specific website or store view needs a
different default, keyed by its **code** (not its ID):

```xml
<config ...>
    <default>
        <section_id>
            <group_id>
                <field_active>0</field_active>
            </group_id>
        </section_id>
    </default>

    <stores>
        <store_view_code>
            <section_id>
                <group_id>
                    <field_active>1</field_active>
                </group_id>
            </section_id>
        </store_view_code>
    </stores>
</config>
```

This keeps per-store behaviour in version control instead of relying
on somebody remembering to set it in the Admin panel of each
environment. A scoped default only takes effect if the field is
actually readable at that scope, so keep it aligned with the
`showInWebsite` / `showInStore` flags from Step 1.

## Step 3: Declare paths and getters in the API interface

**File:** `app/code/Vendor/Module/Api/SystemConfigInterface.php`

This is the core of the pattern. The interface is the single place
that maps a config path to a meaningful, typed accessor. Consumers
depend on this interface, so a path change never propagates beyond
this file.

```php
...
namespace Vendor\Module\Api;

interface SystemConfigInterface
{
    /**
     * @var string
     */
    const XML_PATH_FIELD_ACTIVE = 'section_id/group_id/field_active';

    /**
     * @var string
     */
    const XML_PATH_FIELD_TTL = 'section_id/group_id/field_ttl';

    /**
     * Check if field is active
     *
     * @param  int|null $storeId
     * @return bool
     */
    public function isFieldActive(int $storeId = null): bool;

    /**
     * Get field TTL
     *
     * @param  int|null $storeId
     * @return int
     */
    public function getFieldTtl(int $storeId = null): int;
}
```

Guidelines:

- Every path constant is named `XML_PATH_` + a descriptive name.
  The field ID is the usual choice, but it is not a rule: combine
  the group and field name (or anything clearer) whenever the field
  ID alone would be ambiguous inside the interface —
  `XML_PATH_ROUTES_CACHE_TTL` reads better than `XML_PATH_TTL`.
- Each constant gets its own `/** @var string */` docblock.
- Declare constants first, in the same order the fields appear in
  `system.xml`, then the getters in that same order.
- Name getters after **meaning**, not after the config field:
  booleans read as `isX()` / `hasX()` / `canX()`, values read as
  `getX()`.
- Every method takes an optional `int $storeId = null` and returns a
  concrete type (`bool`, `int`, `string`, `array`). Callers should
  never have to cast a config value themselves.
- When a value needs interpretation (a comma-separated list becoming
  an array, a serialized value being decoded, a default fallback),
  that logic belongs in this manager too, so consumers only ever see
  the final, usable value.

## Step 4: Implement the model

**File:** `app/code/Vendor/Module/Model/System/Config.php`

The implementation always lives in `Model/System/Config.php`, so it
is found in the same place in every module.

```php
...
namespace Vendor\Module\Model\System;

use Magento\Framework\App\Config\ScopeConfigInterface;
use Magento\Store\Model\ScopeInterface;
use Vendor\Module\Api\SystemConfigInterface;

class Config implements SystemConfigInterface
{
    /**
     * @var ScopeConfigInterface
     */
    protected ScopeConfigInterface $_scopeConfig;

    /**
     * Constructor
     *
     * @param ScopeConfigInterface $scopeConfig
     */
    public function __construct(ScopeConfigInterface $scopeConfig)
    {
        $this->_scopeConfig = $scopeConfig;
    }

    /**
     * @inheritDoc
     */
    public function isFieldActive(int $storeId = null): bool
    {
        return (bool) $this->_getConfigData(
            self::XML_PATH_FIELD_ACTIVE, $storeId
        );
    }

    /**
     * @inheritDoc
     */
    public function getFieldTtl(int $storeId = null): int
    {
        return (int) $this->_getConfigData(
            self::XML_PATH_FIELD_TTL, $storeId
        );
    }

    /**
     * Retrieve information from system configuration
     *
     * @param  string   $field
     * @param  int|null $storeId
     * @return mixed
     */
    protected function _getConfigData(string $field, int $storeId = null)
    {
        return $this->_scopeConfig->getValue(
            $field,
            ScopeInterface::SCOPE_STORE,
            $storeId
        );
    }
}
```

Guidelines:

- The protected `_getConfigData()` helper is mandatory: it is the
  only place that talks to `ScopeConfigInterface`, so the scope used
  to read every field is decided once. Reading at
  `ScopeInterface::SCOPE_STORE` with a `null` store falls back to
  the current store and then to the default scope, which is the
  behaviour wanted almost every time.
- Public getters stay one-liners: call `_getConfigData()` with the
  constant and cast the result to the declared return type. Keeping
  them trivial is what makes the interface readable as a catalogue
  of the module's settings.
- Use `@inheritDoc` on the public getters — the interface already
  documents them, and duplicating the docblock invites drift.
- Keep the constant and `$storeId` on the same argument line
  (`self::XML_PATH_X, $storeId`) so a getter stays three lines.

## Step 5: Register the preference

**File:** `app/code/Vendor/Module/etc/di.xml`

```xml
<?xml version="1.0"?>
...
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <preference for="Vendor\Module\Api\SystemConfigInterface"
                type="Vendor\Module\Model\System\Config"/>
</config>
```

Keep `type` aligned under `for` on its own line, matching the
attribute-per-line style used in `system.xml`.

## Step 6: Consume the manager, never `ScopeConfigInterface`

Any class that needs a configuration value injects
`SystemConfigInterface` and calls the descriptive getter:

```php
...
use Vendor\Module\Api\SystemConfigInterface;

class SomeService
{
    /**
     * @var SystemConfigInterface
     */
    protected SystemConfigInterface $_systemConfig;

    ...

    public function doSomething(int $storeId = null): void
    {
        if (!$this->_systemConfig->isFieldActive($storeId)) {
            return;
        }

        $ttl = $this->_systemConfig->getFieldTtl($storeId);
        ...
    }
}
```

If a config path string or a `ScopeConfigInterface::getValue()` call
appears anywhere outside `Model/System/Config.php`, move it into the
manager instead — that is the whole point of the pattern.
