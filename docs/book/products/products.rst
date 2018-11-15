.. index::
   single: Products

Products
========

The **Product** model represents unique products in your Sylius store.
Every product can have different variations and attributes.

.. warning::

   Each product has to have at least one variant to be sold in the shop.

How to create a Product?
------------------------

Before we learn how to create products that can be sold, let's see how to create a product without its complex dependencies.

.. code-block:: php

     /** @var ProductFactoryInterface $productFactory **/
     $productFactory = $this->get('sylius.factory.product');

     /** @var ProductInterface $product */
     $product = $productFactory->createNew();

Creating an empty product is not enough to save it in the database. It needs to have a ``name``, a ``code`` and a ``slug``.

.. code-block:: php

     $product->setName('T-Shirt');
     $product->setCode('00001');
     $product->setSlug('t-shirt');

     /** @var RepositoryInterface $productRepository */
     $productRepository = $this->get('sylius.repository.product');

     $productRepository->add($product);

After being added via the repository, your product will be in the system. But the customers won't be able to buy it.

Variants
========

**ProductVariant** represents a unique kind of product and can have its own pricing configuration, inventory tracking etc.

Variants may be created out of Options of the product, but you are also able to use product variations system without the Options at all.

Virtual Product Variants, that do not require shipping
------------------------------------------------------

.. tip::

     On the ProductVariant there is the possibility to make a product virtual - by setting its ``shippingRequired`` property to ``false``.
     This can be used for non-physical products that for example will be downloadable or installable.

How to create a Product with a Variant?
---------------------------------------

You may need to sell a product in different Variants - for instance, you may have a book, both as hardcover and paperback.
Just like before, to create the product use its factory and save it with the repository.
After that, use the ProductVariantFactory to create a variant of your product.

.. code-block:: php

     /** @var ProductVariantFactoryInterface $productVariantFactory **/
     $productVariantFactory = $this->get('sylius.factory.product_variant');

     /** @var ProductVariantInterface $productVariant */
     $productVariant = $productVariantFactory->createNew();

Having created a Variant, provide it with the required attributes and attach it to your Product.

.. code-block:: php

     $productVariant->setName('Hardcover');
     $productVariant->setCode('1001');
     $productVariant->setPosition(1);
     $productVariant->setProduct($product);

Finally save your Variant in the database using the repository.

.. code-block:: php

     /** @var RepositoryInterface $productVariantRepository */
     $productVariantRepository = $this->get('sylius.repository.product_variant');

     $productVariantRepository->add($variant);

Options
=======

In many cases, you will want to have product with different variations.
The simplest example would be a piece of clothing, like a T-Shirt, available in different sizes and colors
or a glass available in different shapes or colors.
In order to automatically generate appropriate variants, you need to define options.

Every option type is represented by **ProductOption** and references multiple **ProductOptionValue** entities.

For example you can have two options - Size and Color. Each of them will have their own values.

* Size
    * S
    * M
    * L
    * XL
    * XXL

* Color
    * Red
    * Green
    * Blue

After defining possible options for a product let's move on to **Variants** which are in fact combinations of options.

How to create a Product with Options and Variants?
--------------------------------------------------

Firstly let's learn how to prepare an exemplary Option and its values.

.. code-block:: php

     /** @var ProductOptionInterface $option */
     $option = $this->get('sylius.factory.product_option')->createNew();
     $option->setCode('t_shirt_color');
     $option->setName('T-Shirt Color');

     // Prepare an array with values for your option, with codes, locale code and option values.
     $valuesData = [
         'OV1' => ['locale' => 'en_US', 'value' => 'Red'],
         'OV2' => ['locale' => 'en_US', 'value' => 'Blue'],
         'OV3' => ['locale' => 'en_US', 'value' => 'Green'],
     ];

     foreach ($valuesData as $code => $values) {
         /** @var ProductOptionValueInterface $optionValue */
         $optionValue = $this->get('sylius.factory.product_option_value')->createNew();

         $optionValue->setCode($code);
         $optionValue->setFallbackLocale($values['locale']);
         $optionValue->setCurrentLocale($values['locale']);
         $optionValue->setValue($values['value']);

         $option->addValue($optionValue);
     }

After you have created an Option and you keep it as ``$option`` variable let's add it to the Product and generate **Variants**.

.. code-block:: php

     // Assuming that you have a basic product let's add the previously created option to it.
     $product->addOption($option);

     // Having options of a product you can generate variants. Sylius has a service for that operation.
     /** @var ProductVariantGeneratorInterface $variantGenerator */
     $variantGenerator = $this->get('sylius.generator.product_variant');

     $variantGenerator->generate($product);

     // And finally add the product, with its newly generated variants to the repository.
     /** @var RepositoryInterface $productRepository */
     $productRepository = $this->get('sylius.repository.product');

     $productRepository->add($product);

Learn more:
-----------

* :doc:`Product - Bundle Documentation </components_and_bundles/bundles/SyliusProductBundle/index>`
* :doc:`Product - Component Documentation </components_and_bundles/components/Product/index>`
