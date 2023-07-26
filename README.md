# Модульная структура Symfony проекта

Данный структура демонстрирует, как можно организовать модульность в проекте на базе компонента Symfony Dependency Injection.

## Структура каталогов и файлов внутри модулей

```text
web-eshop/
├─ ...
├─ src/
|  ├─ ModuleA/
│  │  ├─ Builder/         ---> Классы строителей, использующие шаблон проектирования `Строитель (Builder)`
│  │  ├─ Command/         ---> Классы консольных команд, наследуемых от класса `Symfony\Component\Console\Command\Command`
│  │  ├─ Controller/      ---> Классы контроллеров, наследуемых от класса `Symfony\Bundle\FrameworkBundle\Controller\AbstractController`
│  │  ├─ Dto/             ---> Классы Data Transfer Object'ов, использующие шаблон проектирования `Data Transfer Object`
│  │  ├─ Entity/          ---> Классы сущностей, которые хранятся в БД
│  │  ├─ Enum/            ---> Классы перечислений (наборы констант). Поскольку наша версия PHP < 8.1, то используем классы/интерфейсы
│  │  ├─ Event/           ---> Классы событий, наследуемых от класса `Symfony\Contracts\EventDispatcher\Event`
│  │  ├─ EventListener/   ---> Классы слушателей событий, помеченных как слушатель событий, например, через атрибут #[AsEventListener]
│  │  ├─ EventSubscriber/ ---> Классы подписчиков событий, реализующих интерфейс `Symfony\Component\EventDispatcher\EventSubscriberInterface`
│  │  ├─ Exception/       ---> Классы исключений, наследуемых от различных классов типа `Exception` или `Error`
│  │  ├─ Factory/         ---> Классы фабрик, использующие различные шаблоны проектирования типа `Фабрика (Factory)`
│  │  ├─ Form/            ---> Классы HTML-форм, наследуемых от класса `Symfony\Component\Form\AbstractType`
│  │  ├─ Repository/      ---> Классы репозиториев, которые выполняют функцию извлечения сущностей из БД
│  │  ├─ Service/         ---> Классы сервисов
│  │  ├─ Twig/            ---> Классы расширений для Twig, наследуемых от класса `Twig\Extension\AbstractExtension`
│  │  ├─ Utils/           ---> Классы вспомогательных утилит
│  │  ├─ ValueObject/     ---> Классы Value Object'ов, которые должны быть неизменяемыми после создания
│  │  ├─ templates/       ---> Файлы шаблонов для Twig
│  │  ├─ di.php           ---> Конфигурация контейнера модуля
│  │  └─ routing.php      ---> Конфигурация маршрутизации модуля
│  ├─ ModuleB/
│  |  └─ ...
│  └─ Kernel.php          ---> Ядро приложения
└─ ...
```

### Описание

Каждый отдельный модуль — это namespace с классами и конфигурациями DI и маршрутизации.
Когда модули все свое "носят" с собой, их легко переименовывать, перегруппировывать, анализировать и удалять.
Особенно удобно в этом случае использовать конфигурацию в формате PHP, тогда рефакторинг в IDE становится полностью
автоматическим.

### Особенности модульной структуры

1. При модульной структуре `config/services.yaml`, `config/services.php`, `config/routes.yaml` и `config/routes.php` не
нужны, так как Symfony Bundles конфигурируются в `config/packages`, а модули проекта конфигурируются в своих `di.php`
и `routing.php`.
2. В `src/Kernel.php` прописан импорт `di.php` и `routing.php` из любых поддиректорий `src`.
3. Модули располагаются в директории `src`, например, `src/ModuleA`. Нужно обратить внимание, что в `di.php`
и `routing.php` также прописан namespace модуля — благодаря этому PhpStorm будет переносить его вслед за классами во
время рефакторинга.

### Структура основных файлов конфигурации

#### Структура файла `Kernel.php`

```php
<?php

declare(strict_types=1);

namespace App;

use Symfony\Bundle\FrameworkBundle\Kernel\MicroKernelTrait;
use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;
use Symfony\Component\HttpKernel\Kernel as BaseKernel;
use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

final class Kernel extends BaseKernel
{
    use MicroKernelTrait;

    protected function configureContainer(ContainerConfigurator $container): void
    {
        $container->import('../config/{packages}/*.yaml');
        $container->import("../config/{packages}/$this->environment/*.yaml");

        $container->import('./**/{di}.php');
        $container->import("./**/{di}_$this->environment.php");
    }

    protected function configureRoutes(RoutingConfigurator $routes): void
    {
        $routes->import('../config/{routes}/*.yaml');
        $routes->import("../config/{routes}/$this->environment/*.yaml");

        $routes->import('./**/{routing}.php');
        $routes->import("./**/{routing}_$this->environment.php");
    }
}
```

#### Структура файла `di.php`

```php
<?php

declare(strict_types=1);

namespace App\ModuleA;

use Symfony\Component\DependencyInjection\Loader\Configurator\ContainerConfigurator;

return static function (ContainerConfigurator $container): void {
    $services = $container
        ->services()
        ->defaults()
        ->autowire()
        ->autoconfigure();

    $services->load('App\\ModuleA\\', __DIR__ . '/{Controller}')
        ->tag('controller.service_arguments');

    $services->load('App\\ModuleA\\', __DIR__ . '/{Service}');

    $container->extension('twig', [
        'paths' => [
            __DIR__ . '/templates' => 'ModuleA',
        ],
    ]);
};
```

#### Структура файла `routing.php`

```php
<?php

declare(strict_types=1);

namespace App\ModuleA;

use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

return static function (RoutingConfigurator $routes): void {
    $routes->import(
        '../src/Controller/',
        'annotation',
    );
};
```
