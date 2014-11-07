Компоненты
=========

Компоненты — это главные строительные блоки приложений основанных на Yii. Компоненты наследуются от класса
[[yii\base\Component]] или его наследников. Три главные возможности, которые компоненты предоставляют для других классов:

* [Свойства](concept-properties.md).
* [События](concept-events.md).
* [Поведения](concept-behaviors.md).

Как по отдельности, так и вместе, эти возможности делают классы Yii более простыми в настройке и использовании.
Например, пользовательские компоненты, включающие в себя [[yii\jui\DatePicker|виджет выбора даты]], могут быть
использованы в [представлении](structure-view.md) для генерации интерактивных элементов выбора даты:

```php
use yii\jui\DatePicker;

echo DatePicker::widget([
    'language' => 'ru',
    'name'  => 'country',
    'clientOptions' => [
        'dateFormat' => 'yy-mm-dd',
    ],
]);
```

Свойства виджета легко доступны для записи потому, что его класс унаследован от класса [[yii\base\Component]].

Компоненты — очень мощный инструмент. Но в то же время они немного тяжелее обычных объектов, потому что на поддержку
[событий](concept-events.md) и [поведений](concept-behaviors.md) тратится дополнительные память и процессорное время.
Если ваши компоненты не нуждаются в этих двух возможностях, вам стоит унаследовать их от [[yii\base\Object]],
а не от [[yii\base\Component]]. Поступив так, вы сделаете ваши компоненты такими же эффективными, как и обычные PHP объекты,
но с поддержкой [свойств](concept-properties.md).

При наследовании ваших классов от [[yii\base\Component]] или [[yii\base\Object]], рекомендуется следовать некоторым
соглашениям:

- Если вы переопределяете конструктор, то добавьте *последним* аргументом параметр `$config` и затем передайте его
  в конструктор предка.
- Всегда вызывайте конструктор предка *в конце* вашего переопределенного конструктора.
- Если вы переопределяете метод [[yii\base\Object::init()]], убедитесь, что вы вызываете родительскую реализацию этого
  метода *в начале* вашего метода `init()`.

Пример:

```php
namespace yii\components\MyClass;

use yii\base\Object;

class MyClass extends Object
{
    public $prop1;
    public $prop2;

    public function __construct($param1, $param2, $config = [])
    {
        // ... инициализация происходит перед тем, как будет применена конфигурация.

        parent::__construct($config);
    }

    public function init()
    {
        parent::init();

        // ... инициализация происходит после того, как была применена конфигурация.
    }
}
```

Следуя этому руководству вы позволите [настраивать](concept-configurations.md) ваш компонент при создании. Например:

```php
$component = new MyClass(1, 2, ['prop1' => 3, 'prop2' => 4]);
// альтернативный способ
$component = \Yii::createObject([
    'class' => MyClass::className(),
    'prop1' => 3,
    'prop2' => 4,
], [1, 2]);
```

> Информация: Способ инициализации через вызов [[Yii::createObject()]] выглядит более сложным. Но в то же время он более
  мощный из-за того, что он реализован на самом верху [контейнера внедрения зависимостей](concept-di-container.md).

Жизненый цикл объектов класса [[yii\base\Object]] содержит следующие этапы:

1. Предварительная инициализация в конструкторе. Здесь вы можете установить значения свойств по умолчанию.
2. Конфигурация объекта с помощью `$config`. Во время конфигурации могут быть перезаписаны значения свойств по умолчанию,
   установленные в конструкторе.
3. Конфигурация после инициализации в методе [[yii\base\Object::init()|init()]]. Вы можете переопределить этот метод,
   для проверки готовности объекта и нормализации свойств.
4. Вызов методов объекта.

Первые три шага всегда выполняются из конструктора объекта. Это значит, что если вы получите экземпляр объекта, он уже
будет проинициализирован и готов к работе.