# Design Patterns indispensables pour un développeur PHP/Symfony

Un design pattern est une solution réutilisable à un problème récurrent de conception.

Les patterns ne sont pas du code à "copier-coller" sans trop réfléchir, mais des structures logiques applicables à différents contextes.

Ils sont regroupés en 3 familles :
- Créationnels
- Structurels
- Comportementaux

# Table des matières
1. [Les patterns de création](#les-patterns-de-création)
    1. [Pattern Singleton](#pattern-singleton)
    2. [Pattern Factory](#pattern-factory)
2. [Les patterns structurels](#les-patterns-structurels)
    1. [Pattern Adapter](#pattern-adapter)
    2. [Pattern Decorator](#pattern-decorator)
3. [Les Patterns comportementaux](#les-patterns-comportementaux)
    1. [Pattern Strategy](#pattern-strategy)
    2. [Pattern Observer](#pattern-observer)
4. [Usage des Design Patterns dans Symfony](#usage-des-design-patterns-dans-symfony)

## Les patterns de création ([Doc complète](https://refactoring.guru/fr/design-patterns/creational-patterns))

### Pattern Singleton
Garantit qu’une classe n’a qu’une seule instance et fournit un point d’accès global.

⚠️ À utiliser avec parcimonie (risque d’effet global).
```php
// La classe Logger est instancié à l'aide d'un singleton
class Logger {
    private static ?Logger $instance = null;

    private function __construct() {}

    public static function getInstance(): Logger {
        return self::$instance ??= new Logger();
    }

    public function log(string $msg) {
        echo $msg;
    }
}

```

### Pattern Factory

Délègue la création d’un objet à une méthode d’une sous-classe.

```php
interface Transport {
    public function deliver(): string;
}

class Truck implements Transport {
    public function deliver(): string {
        return "Livraison par camion.";
    }
}

class TransportFactory {
    public static function create(string $type): Transport {
        return match ($type) {
            'truck' => new Truck(),
            default => throw new Exception("Type inconnu"),
        };
    }
}

```
## Les patterns structurels ([Doc complète](https://refactoring.guru/fr/design-patterns/structural-patterns))

### Pattern Adapter

Permet de faire communiquer deux interfaces à priori incompatibles.

```php
class OldSystem {
    public function request(): string {
        return "Données dans ancien format";
    }
}

class NewSystem {
    public function get(): string {
        return "Données au nouveau format";
    }
}

class Adapter extends NewSystem {
    private OldSystem $old;

    public function __construct(OldSystem $old) {
        $this->old = $old;
    }

    public function get(): string {
        return $this->old->request(); // adaptation ici
    }
}

```

### Pattern Decorator
Ajoute dynamiquement des responsabilités à un objet.

```php
interface Notifier {
    public function send(string $message): void;
}

class EmailNotifier implements Notifier {
    public function send(string $message): void {
        echo "Envoi par email : $message";
    }
}

class SlackNotifier implements Notifier {
    public function __construct(private Notifier $wrapped) {}

    public function send(string $message): void {
        $this->wrapped->send($message);
        echo " + Envoi Slack : $message";
    }
}

```

## Les Patterns comportementaux ([Doc complète](https://refactoring.guru/fr/design-patterns/behavioral-patterns))

### Pattern Strategy
Permet de changer dynamiquement l’algorithme utilisé par une classe.

```php
interface TaxStrategy {
    public function calculate(float $price): float;
}

class FranceTax implements TaxStrategy {
    public function calculate(float $price): float {
        return $price * 1.2;
    }
}

class Cart {
    public function __construct(private TaxStrategy $tax) {}

    public function total(float $price): float {
        return $this->tax->calculate($price);
    }
}

```

### Pattern Observer

Permet de notifier plusieurs objets lorsqu’un changement d’état survient.

```php
interface Observer {
    public function update(string $event): void;
}

class LoggerObserver implements Observer {
    public function update(string $event): void {
        echo "Log : $event\n";
    }
}

class EventManager {
    private array $observers = [];

    public function attach(Observer $observer): void {
        $this->observers[] = $observer;
    }

    public function notify(string $event): void {
        foreach ($this->observers as $obs) {
            $obs->update($event);
        }
    }
}

```

N.B : Symfony utilise de nombreux design patterns :

- Dependency Injection = Strategy + Singleton

- EventDispatcher = Observer

- HttpKernel = Front Controller

- Form Component = Decorator

- Messenger = Command, Event, CQRS
- ...

