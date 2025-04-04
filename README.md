# Design Patterns indispensables pour un développeur PHP/Symfony

## Table des matières
1. [Les patterns de création](#les-patterns-de-création)
    1. [Pattern Singleton](#pattern-singleton)
    2. [Pattern Factory](#pattern-factory)
2. [Les patterns structurels](#les-patterns-structurels)
    1. [Pattern Adapter](#pattern-adapter)
    2. [Pattern Decorator](#pattern-decorator)
3. [Les Patterns comportementaux](#les-patterns-comportementaux)
    1. [Pattern Strategy](#pattern-strategy)
    2. [Pattern Observer](#pattern-observer)
    3. [Pattern Command](#pattern-command)
    4. [Pattern Front Controller](#pattern-front-controller)
4. [BONUS : comment Symfony les utilise-t-il ?](#bonus--comment-symfony-les-utilise-t-il-)

Un design pattern est une solution réutilisable à un problème récurrent de conception.

Les patterns ne sont pas du code à "copier-coller" sans trop réfléchir, mais des structures logiques applicables à différents contextes.

Ils sont regroupés en 3 familles :
- Créationnels
- Structurels
- Comportementaux

## Les patterns de création ([Doc complète](https://refactoring.guru/fr/design-patterns/creational-patterns))

### Pattern Singleton
Garantit qu’une classe n’a qu’une seule instance et fournit un point d’accès global.

Le pattern Singleton permet de dire : “Cette chose, on ne l’instancie qu’une seule fois dans tout le programme, et tout le monde utilise la même instance”

*Imagine un chef d’orchestre. Tu ne peux pas en avoir deux en même temps dans une même salle de concert. Sinon, c’est le bazar.*

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

**Cas d'utilisation :** Le système de connexion à une base de données. Tu n’as pas besoin de créer 10 connexions à la base, tu en ouvres une et tout le monde l’utilise

### Pattern Factory

Délègue la création d’un objet à une méthode d’une sous-classe en fonction des besoins. Toi, tu ne sais pas comment c’est fabriqué à l’intérieur, tu fais juste ta commande (comme lorsque tu vas en magasin, tu n'as aucune idée de comment ton produit préféré a été fait en usine, en principe 😜 ).

L'idée avec le pattern Factory est de pouvoir créer des objets sans te soucier de leurs détails de fabrication.

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

Imagine que tu aies un chargeur avec une prise française mais que ta prise murale est anglaise (ou Suisse comme cela m'est arrivé la première fois que j'ai été dans ce beau pays). Tu utilises un adaptateur pour que ça fonctionne.

Dans le dev c'est pareil : parfois deux systèmes ne parlent pas la même "langue" (interface), mais tu veux quand même qu’ils collaborent. 

L’adapter fait le pont entre les deux.

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
**Cas d'utilisation :** : Tu as un ancien système (vieille bibliothèque PHP) qui ne respecte pas les conventions modernes, tu crées un adapter pour l’utiliser dans ton nouveau code basé sur Symfony.

### Pattern Decorator

Le decorator permet de prendre un objet simple, et d’y ajouter des fonctionnalités sans le modifier fondamentalement parlant.

Imagine que tu commandes un café. Tu peux le prendre nature, ou ajouter du lait, du sucre, de la crème chantilly. À chaque fois, c’est le même café… mais avec un truc en plus.

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

**Cas d'utilisation :** : Tu as un service de notification de base (email), et tu veux ensuite qu’il envoie aussi sur Slack ou SMS. Plutôt que de réécrire tout, tu "décore" le service de base.

## Les Patterns comportementaux ([Doc complète](https://refactoring.guru/fr/design-patterns/behavioral-patterns))

### Pattern Strategy

Permet de changer dynamiquement l’algorithme utilisé par une classe.

L'idée étant que si tu veux exécuter une action, tu vas pouvoir choisir comment le faire (et pouvoir changer facilement).

Imageons un peu tout ça : tu veux aller au travail. Tu peux y aller en voiture, en métro, ou à vélo. Le but est le même (arriver au boulot et serrer la main tes collègues en physique ou en virtuel), mais la stratégie change selon tes préférences ou la météo.

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

**Cas d'utilisation :** : Calcul des taxes - tu peux avoir une stratégie différente selon le pays du client. France, Belgique, Suisse = stratégies différentes.

### Pattern Observer

Permet de notifier plusieurs objets lorsqu’un changement d’état survient. L'idée serait que dans ton code, quand un événement arrive (ex. : un nouvel utilisateur s’inscrit), plusieurs autres parties du système peuvent être averties et réagir (Envoyez un e-mail à l'admin, mettre à jour le CRM distant, ajout à la newsletter, ...)

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

### Pattern Command 

Représente une action à faire, qu’on peut envoyer à quelqu’un (comme un agent), qui va s’en charger.

```php
// La commande

// src/Message/SendWelcomeEmail.php

namespace App\Message;

class SendWelcomeEmail
{
    public function __construct(
        private string $email,
        private string $name
    ) {}

    public function getEmail(): string {
        return $this->email;
    }

    public function getName(): string {
        return $this->name;
    }
}

```

```php
// Le Handler

// src/MessageHandler/SendWelcomeEmailHandler.php

namespace App\MessageHandler;

use App\Message\SendWelcomeEmail;
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mime\Email;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;

#[AsMessageHandler]
class SendWelcomeEmailHandler
{
    public function __construct(private MailerInterface $mailer) {}

    public function __invoke(SendWelcomeEmail $message): void
    {
        $email = (new Email())
            ->from('hello@example.com')
            ->to($message->getEmail())
            ->subject('Bienvenue !')
            ->text("Bonjour {$message->getName()}, bienvenue sur notre site.");

        $this->mailer->send($email);
    }
}

```

Envoi dans un Controleur / Service 

```php
use App\Message\SendWelcomeEmail;
use Symfony\Component\Messenger\MessageBusInterface;

// ...
$bus->dispatch(new SendWelcomeEmail($user->getEmail(), $user->getName()));

```

### Pattern Front Controller

C'est la porte d'entrée unique. C'est comme un immeuble avec plein d’appartements… mais une seule porte d’entrée, avec un interphone qui redirige les gens au bon endroit.

Symfony et beaucoup d'autres Frameworks (sinon tous) fonctionnent comme ça. Toute requête HTTP arrive par un seul fichier : index.php, puis il redirige vers le bon contrôleur, la bonne action.

```php
// public/index.php (Old fashion Symfony FC)
use App\Kernel;
use Symfony\Component\ErrorHandler\Debug;
use Symfony\Component\HttpFoundation\Request;

require dirname(__DIR__).'/vendor/autoload.php';

Debug::enable();

$kernel = new Kernel('dev', true);
$request = Request::createFromGlobals();

// TOUT commence ici
$response = $kernel->handle($request);
$response->send();

$kernel->terminate($request, $response);

```

## BONUS : comment Symfony les utilise-t-il ?

| Pattern  | Où dans Symfony ? |
| -------- | ------- |
| Singleton  | Services injectés (ex : Logger) |
| Factory  | FormFactory, Serializer, etc. |
| Adapter  | Cache adapters (ex : Redis, Filesystem) |
| Decorator  | Security voters, Middlewares |
| Strategy  | Authentification (Token, JWT, etc.) |
| Observer  | EventDispatcher |
| Command  | Symfony Console & Messenger |
| Front Controller  | index.php dans public/ |

