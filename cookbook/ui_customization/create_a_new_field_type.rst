How to add a new field type
===========================

If you want a custom rendering for one of your attributes, attribute types or reference data you need to create a new field type. In this cookbook, we will go through each step needed to create a custom field type.

Let's say we want to have a slider to represent each "number attribute" having a minimum and a maximum value limit.

Before diving into code, we need to understand what's going on under the hood:

 - In akeneo, we have attributes with properties and an attribute type. In 1.3 the rendering of an attribute was driven by its attribute type. In 1.4 we introduced a field provider.
 - This field provider gets an attribute and returns a field type
 - In the `form_extensions.yml` or in the `form_extensions` folder of your `Resources/config` bundle's folder you can map this field to the actual requirejs module
 - The requirejs module contains the field's logic


Here is a representation of this architecture:

.. image:: field_process.png

Create a field provider
+++++++++++++++++++++++

To create a custom field, first we need to create a FieldProvider for our new field type:

.. code-block:: php
    :linenos:

    <?php

    namespace Acme\Bundle\CustomBundle\Enrich\Provider\Field;

    use Pim\Component\Catalog\Model\AttributeInterface;
    use Pim\Bundle\EnrichBundle\Provider\Field\FieldProviderInterface;

    class RangeFieldProvider implements FieldProviderInterface
    {
        /**
         * {@inheritdoc}
         */
        public function getField($attribute)
        {
            return 'acme-range-field';
        }

        /**
         * {@inheritdoc}
         */
        public function supports($element)
        {
            //We only support number fields that have a number min and max property
            return $element instanceof AttributeInterface &&
                $element->getAttributeType() === 'pim_catalog_number' &&
                null !== $element->getNumberMin() &&
                null !== $element->getNumberMax();
        }
    }


Next, you need to register it in your service.yml file:

.. code-block:: yaml
    :linenos:

    parameters:
        acme.custom.provider.field.range.class: Acme\Bundle\CustomBundle\Enrich\Provider\Field\RangeFieldProvider

    services:
        acme.custom.provider.field.range:
            class: %acme.custom.provider.field.range.class%
            tags:
                - { name: pim_enrich.provider.field, priority: 90 }


Your field provider is now registered, congrats!

Create the form field
+++++++++++++++++++++

Now that we have a field provider, we can create the field itself:

.. code-block:: javascript
    :linenos:

    /*
     * src/view/product/field/range.tsx
     */
    import * as React from 'react';
    import { Value } from 'pim/model/product/value';
    import { Attribute } from 'pim/model/catalog/attribute';

    export default (
      { value, attribute, onFieldChange }:
      { value: Value, attribute: Attribute, onFieldChange: any }
    ) => {
      return <input type="range" min={attribute.number_min} max={attribute.number_max} value={ value.data || '' }
        onChange={ (event: any) => { fieldChanged(event, onFieldChange, value, attribute) }  }
        data-field={ attribute.code }
        data-locale={ value.locale }
        data-scope={ value.scope }
      />;
    }

    const fieldChanged = (event: any, onFieldChange: any, value: Value, attribute: Attribute) => {
      const data = event.currentTarget.value;

      onFieldChange(Object.assign({}, value, {data}), attribute);
    }

You can now register this file into your module configuration:

.. code-block:: json
    :linenos:

    // src/config/modules.json
    {
      "view/product/field/range": "acme/view/product/field/range"
    }

Next you can register the module into the view configuration file:

.. code-block:: json
    :linenos:

    // src/config/views.json
    [
      {
        "code": "acme/product/edit/tabs/attributes/field/range",
        "view": "view/product/field/range",
        "parent": "pim/product/edit/tabs/attributes/fields/field",
        "section": "fields",
        "attribute_type": "acme-range-field"
      }
    ]

Now you need to declare those files in your package.json:

.. code-block:: json
    :linenos:

    "akeneo-pim": {
      "views": "src/config/views.json",
      "modules-mapping": "src/config/modules.json"
    }

Once you have done that you can recompile the configuration:

.. code-block:: bash
    :linenos:

    node ./node_modules/.bin/pim-discover-modules
    node ./node_modules/.bin/pim-compile-modules

You can now relaunch your webpack server and reload the product edit page.
