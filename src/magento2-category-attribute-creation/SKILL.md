---
name: magento2-category-attribute-creation
description: >
  Use this skill every time you need to create
  a category attribute in a Magento 2 project
---

# Goal

Add a custom attribute to the category entity
inside a Magento 2 project.
Optionally, add it to the category form used 
in the Magento Admin panel.

## Step 1: Create a source model (optional)

If your attribute is a dropdown or multiselect, you need a source model to provide the options.

**File:** `app/code/Vendor/Module/Model/Catalog/Category/Attribute/Source/Attribute.php`

```php
...
namespace Vendor\Module\Model\Catalog\Category\Attribute\Source;

use Magento\Eav\Model\Entity\Attribute\Source\AbstractSource;

class Attribute extends AbstractSource
{
    ...
    public function getAllOptions()
    {
        if (!$this->_options) {
            /**
             * @note To show the empty option in admin panel,
             *       it is required to add a label 
             *       with an space instead
             *       of an empty string
             */
            $this->_options = [
                [
                    'label' => ' ',
                    'value' => ''
                ],
                [
                    'label' => __('Option 1'), 
                    'value' => 'option_1'
                ],
                [
                    'label' => __('Option 2'), 
                    'value' => 'option_2'
                ],
            ];
        }
        
        return $this->_options;
    }
}
```

## Step 2: Create a data patch to add the attribute

Data patches are the modern way (since Magento 2.3) 
to add or modify EAV attributes.

**File:** `app/code/Vendor/Module/Setup/Patch/Data/AddCustomCategoryAttribute.php`

```php
...
namespace Vendor\Module\Setup\Patch\Data;

use Magento\Catalog\Model\Category;
use Magento\Catalog\Setup\CategorySetupFactory;
use Magento\Framework\Setup\ModuleDataSetupInterface;
use Magento\Framework\Setup\Patch\DataPatchInterface;
use Magento\Eav\Model\Entity\Attribute\ScopedAttributeInterface;
use Vendor\Module\Model\Catalog\Category\Attribute\Source\Attribute;

class AddCustomCategoryAttribute implements DataPatchInterface
{
    ...
    public function apply()
    {
        $categorySetup = $this->_categorySetupFactory->create(
            ['setup' => $this->_moduleDataSetup]
        );

        $categorySetup->addAttribute(Category::ENTITY, 'custom_attribute_code', [
            'type'                  => 'varchar', // Data type in DB
            'label'                 => 'Custom Attribute Label',
            'input'                 => 'select',  // UI input type
            'source'                => Attribute::class,
            'required'              => false,
            'sort_order'            => 100,
            'global'                => ScopedAttributeInterface::SCOPE_STORE,
            'group'                 => 'General Information', // Tab name in admin
            'is_used_in_grid'       => false,
            'is_visible_in_grid'    => false,
            'is_filterable_in_grid' => false,
        ]);
    }
    ...
}
```

## Step 3: Add the field to the admin category form

Category forms use UI Components. 
You must extend the `category_form.xml` to make the field visible.

**File:** `app/code/Vendor/Module/view/adminhtml/ui_component/category_form.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
...
<form xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
      xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Ui:etc/ui_configuration.xsd">
    <fieldset name="general"> <!-- Or the name of the fieldset/group defined in the data patch -->
        <field name="custom_attribute_code" sortOrder="100" formElement="select">
            <settings>
                <dataType>string</dataType>
                <label translate="true">Custom Attribute Label</label>
            </settings>
        </field>
    </fieldset>
</form>
```