.. index::
   single: Dependency Injection; Parameters

Introduction aux paramètres
===========================

Vous pouvez définir des paramètres dans le conteneur de services qui peuvent être
directement utilisé ou en partie dans les définitions des services. Cela peut
aider à séparer les valeurs que vous changez régulièrement.

Récupérer et définir les paramètres du Conteneur
------------------------------------------------

Travailler avec les paramètres du conteneur est très facile si vous utilisez
les méthodes d'accès du conteneur pour les paramètres. Vous pouvez contrôler
qu'un paramètre a été défini dans le conteneur avec::

     $container->hasParameter('mailer.transport');

Vous pouvez récupérer des paramètres définis dans le conteneur avec::

    $container->getParameter('mailer.transport');

et définir un paramètre dans le conteneur grâce à::

    $container->setParameter('mailer.transport', 'sendmail');

.. note::

    Vous ne pouvez définir un paramètre qu'avant que le conteneur soit compilé.
    Pour en apprendre plus sur la compilation du conteneur, lisez
    :doc:`/components/dependency_injection/compilation`.

Paramètres dans le fichier de configuration
-------------------------------------------

Vous pouvez aussi utiliser la section ``parameters`` du fichier de configuration
pour définir des paramètres:

.. configuration-block::

    .. code-block:: yaml

        parameters:
            mailer.transport: sendmail

    .. code-block:: xml

        <parameters>
            <parameter key="mailer.transport">sendmail</parameter>
        </parameters>

    .. code-block:: php

        $container->setParameter('mailer.transport', 'sendmail');

Vous pouvez récupérer les valeurs des paramètres directement dans le conteneur,
vous pouvez aussi les utiliser dans les fichiers de configuration. Vous pouvez
les référencer en les entourant de signe pourcentage, exemple ``%mailer.transport%``.
Vous pouvez les utiliser pour injecter ces valeurs dans vos services. Cela permet
de configurer différentes versions de services entre applications ou plusieurs
services basés sur une même classe, mais configurer différemment dans une seule
application. Vous pouvez injecter le choix du transport du courrier directement
dans la classe ``Mailer``. En créant un paramètre, cela facilite les changements
de ce paramètre plutôt que de les intégrer dans la définition du service.

.. configuration-block::

    .. code-block:: yaml

        parameters:
            mailer.transport: sendmail

        services:
            mailer:
                class:     Mailer
                arguments: ['%mailer.transport%']

    .. code-block:: xml

        <parameters>
            <parameter key="mailer.transport">sendmail</parameter>
        </parameters>

        <services>
            <service id="mailer" class="Mailer">
                <argument>%mailer.transport%</argument>
            </service>
        </services>

    .. code-block:: php

        use Symfony\Component\DependencyInjection\Reference;

        // ...
        $container->setParameter('mailer.transport', 'sendmail');
        $container
            ->register('mailer', 'Mailer')
            ->addArgument('%mailer.transport%');

.. caution::

    The values between ``parameter`` tags in XML configuration files are not
    trimmed.
    Les valeurs entre les tags ``parameter`` dans votre configuration XML ne
    sont pas "trimmée" (les espaces, les retours à la ligne, etc. ne sont pas retirées).

    Cela signifie que la configuration ci-dessous aura comme valeur
    ``\n    sendmail\n``:

    .. code-block:: xml

        <parameter key="mailer.transport">
            sendmail
        </parameter>

    Dans certains cas (pour les constantes ou les noms de classes),
    il se peut que cela génère des erreurs. Pour éviter cela, vous devez toujours
    écrire la définition de vos paramètres sur une ligne :

    .. code-block:: xml

        <parameter key="mailer.transport">sendmail</parameter>


Si vous l'utilisez ailleurs aussi, vous n'aurez besoin que de changer la valeur
du paramètre à un seul endroit.

Vous pouvez également utiliser les paramètres dans la définition du service, par
exemple, en ajoutant le paramètre dans la classe du service:

.. configuration-block::

    .. code-block:: yaml

        parameters:
            mailer.transport: sendmail
            mailer.class: Mailer

        services:
            mailer:
                class:     '%mailer.class%'
                arguments: ['%mailer.transport%']

    .. code-block:: xml

        <parameters>
            <parameter key="mailer.transport">sendmail</parameter>
            <parameter key="mailer.class">Mailer</parameter>
        </parameters>

        <services>
            <service id="mailer" class="%mailer.class%">
                <argument>%mailer.transport%</argument>
            </service>

        </services>

    .. code-block:: php

        use Symfony\Component\DependencyInjection\Reference;

        // ...
        $container->setParameter('mailer.transport', 'sendmail');
        $container->setParameter('mailer.class', 'Mailer');
        $container
            ->register('mailer', '%mailer.class%')
            ->addArgument('%mailer.transport%');

        $container
            ->register('newsletter_manager', 'NewsletterManager')
            ->addMethodCall('setMailer', array(new Reference('mailer')));

.. note::

    Le signe pourcentage dans un paramètre ou un argument, faisant parti de la
    chaîne de caractère, doit être échappé avec un autre signe pourcentage:

    .. configuration-block::

        .. code-block:: yaml

            arguments: ['http://symfony.com/?foo=%%s&bar=%%d']

        .. code-block:: xml

            <argument type="string">http://symfony.com/?foo=%%s&bar=%%d</argument>

        .. code-block:: php

            ->addArgument('http://symfony.com/?foo=%%s&bar=%%d');

.. _component-di-parameters-array:

Tableau de paramètres
---------------------

Les paramètres ne sont pas nécessairement de simple chaîne de caractères, ils peuvent
être aussi des tableaux. Pour le format XML, vous aurez besoin d'utiliser l'attribut
``type="collection"`` pour tous les paramètres qui sont des tableaux.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        parameters:
            my_mailer.gateways:
                - mail1
                - mail2
                - mail3
            my_multilang.language_fallback:
                en:
                    - en
                    - fr
                fr:
                    - fr
                    - en

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <parameters>
            <parameter key="my_mailer.gateways" type="collection">
                <parameter>mail1</parameter>
                <parameter>mail2</parameter>
                <parameter>mail3</parameter>
            </parameter>
            <parameter key="my_multilang.language_fallback" type="collection">
                <parameter key="en" type="collection">
                    <parameter>en</parameter>
                    <parameter>fr</parameter>
                </parameter>
                <parameter key="fr" type="collection">
                    <parameter>fr</parameter>
                    <parameter>en</parameter>
                </parameter>
            </parameter>
        </parameters>

    .. code-block:: php

        // app/config/config.php
        use Symfony\Component\DependencyInjection\Definition;

        $container->setParameter('my_mailer.gateways', array('mail1', 'mail2', 'mail3'));
        $container->setParameter('my_multilang.language_fallback', array(
            'en' => array('en', 'fr'),
            'fr' => array('fr', 'en'),
        ));

.. _component-di-parameters-constants:

Des constantes en paramètres
----------------------------

Le conteneur supporte aussi la définition des constantes PHP comme paramètres.
Pour tirer parti de cette fonctionnalité, il faut mapper le nom de la constant
à une clé de paramètre, et définissez son type comme ``constant``.

.. configuration-block::

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8"?>

        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

            <parameters>
                <parameter key="global.constant.value" type="constant">GLOBAL_CONSTANT</parameter>
                <parameter key="my_class.constant.value" type="constant">My_Class::CONSTANT_NAME</parameter>
            </parameters>
        </container>

    .. code-block:: php

            $container->setParameter('global.constant.value', GLOBAL_CONSTANT);
            $container->setParameter('my_class.constant.value', My_Class::CONSTANT_NAME);

.. note::

    Cela ne fonctionne pas la configuration Yaml. Si vous utilisez Yaml, vous
    pouvez importer un fichier XML pour tirer parti de cette fonctionnalité:

    .. configuration-block::

        .. code-block:: yaml

            # app/config/config.yml
            imports:
                - { resource: parameters.xml }

Mots clés PHP en XML
--------------------

Par défaut, ``true``, ``false`` et ``null`` en XML sont transformés en mots clés PHP
(respectivement ``true``, ``false`` et ``null``) :

.. code-block:: xml

    <parameters>
        <parameter key="mailer.send_all_in_once">false</parameters>
    </parameters>

    <!-- après parsing
    $container->getParameter('mailer.send_all_in_once'); // retourne false
    -->

Pour ne pas avoir cette transformation, utilisez le type ``string`` :

.. code-block:: xml

    <parameters>
        <parameter key="mailer.some_parameter" type="string">true</parameter>
    </parameters>

    <!-- après parsing
    $container->getParameter('mailer.some_parameter'); // retourne "true"
    -->

.. note::

    Ceci n'est pas disponible si vous le faites en YAML ou en PHP, parce que
    ces langages supportent déjà les mots clés PHP.

Syntaxe pour la référence de service
------------------------------------

Vous pouvez bien évidemment référencer vos services, la syntaxe sera un
peu différente pour chacun des formats. Il vous est possible de configurer
leur comportement si le service référencé n'existe pas. Par défaut, une
exception sera levée quand un service référencé est inexistant.

YAML
~~~~

Commencez la chaine de caractère avec ``@`` ou ``@?`` pour référencer un
service en YAML.

* ``@mailer`` référence le service ``mailer``. Si le service n'existe pas, une exception sera levée;
* ``@?mailer`` références le service ``mailer``. Si le service n'existe pas, il sera ignoré;

.. tip::

    Utilisez ``@@`` pour échaper le symbole ``@`` en YAML. ``@@mailer`` sera
    converti en la chaîne de caractères ``"@mailer"`` au lieu de référencer
    le service ``mailer``.

XML
~~~

En XML, utilisez le type ``service``. Le comportement peut être spécifié si
le service n'existe pas en utilisant l'argument ``on-invalid``. Par défaut,
une exception est levée. Les valeurs valides pour l'argument ``on-invalid``
sont ``null`` (utilisez ``null`` au lieu d'un service manquant) ou ``ignored``
(très similaire à ``null``, sauf dans le cas d'un appel de méthode, celui-ci
est supprimé).

PHP
~~~

En PHP, vous pouvez utiliser la classe
:class:`Symfony\\Component\\DependencyInjection\\Reference` pour référencer
un service. Le comportement dinvalidité est configuré en utilisant le second
argument et constantes du constructeur de l'interface
:class:`Symfony\\Component\\DependencyInjection\\ContainerInterface`.
