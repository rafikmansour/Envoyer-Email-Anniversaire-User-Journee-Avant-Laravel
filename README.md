# Controller Laravel pour envoyer un e-mail à un utilisateur un jour avant son anniversaire.

Pour envoyer un e-mail à un utilisateur un jour avant son anniversaire, nous allons créer un contrôleur Laravel qui s'occupera de récupérer les utilisateurs dont l'anniversaire est dans un jour, puis enverra un e-mail à ces utilisateurs. Nous allons également configurer la tâche planifiée (cron job) qui exécutera le contrôleur tous les jours pour vérifier les anniversaires et envoyer les e-mails le cas échéant.

Voici comment créer le contrôleur et la tâche planifiée :

Créez un nouveau contrôleur en utilisant la commande artisan :

**php artisan make:controller BirthdayReminderController**

Ouvrez le fichier BirthdayReminderController.php créé dans le répertoire app/Http/Controllers.

Dans ce fichier, ajoutez les méthodes nécessaires pour récupérer les utilisateurs dont l'anniversaire est dans un jour et envoyer l'e-mail. Nous utiliserons Laravel's Mail pour l'envoi d'e-mail. Voici un exemple de contenu du contrôleur.


<code><pre>
// app/Http/Controllers/BirthdayReminderController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\Mail;
use App\Mail\BirthdayReminderMail;

class BirthdayReminderController extends Controller
{
    public function sendBirthdayReminders()
    {
        // Récupérer les utilisateurs dont l'anniversaire est dans un jour
        $users = User::whereDay('birthday', now()->addDay()->day)
            ->whereMonth('birthday', now()->addDay()->month)
            ->get();

        // Envoyer un e-mail de rappel à chaque utilisateur
        foreach ($users as $user) {
            Mail::to($user->email)->send(new BirthdayReminderMail($user));
        }

        return response()->json(['message' => 'Birthday reminders sent successfully.']);
    }
}

</pre></code>

Ensuite, nous devons créer le modèle User et configurer une migration pour la table d'utilisateurs qui contiendra le champ birthday pour enregistrer la date de naissance de l'utilisateur.

Créez un nouveau mail en utilisant la commande artisan :

**php artisan make:mail BirthdayReminderMail**

Ouvrez le fichier BirthdayReminderMail.php créé dans le répertoire app/Mail.

Dans ce fichier, définissez la logique de l'e-mail de rappel d'anniversaire. Voici un exemple de contenu du mail :

<code>
<pre>
  // app/Mail/BirthdayReminderMail.php

namespace App\Mail;

use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use App\Models\User;

class BirthdayReminderMail extends Mailable
{
    use SerializesModels;

    public $user;

    /**
     * Create a new message instance.
     *
     * @param  User  $user
     * @return void
     */
    public function __construct(User $user)
    {
        $this->user = $user;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->subject('Birthday Reminder')
                    ->view('emails.birthday-reminder');
    }
}

</pre>

</code>


Ensuite, créez une vue pour l'e-mail de rappel d'anniversaire. Créez un nouveau fichier birthday-reminder.blade.php dans le répertoire resources/views/emails.

Dans ce fichier, définissez le contenu de l'e-mail. Par exemple :

<code><pre>
<!-- resources/views/emails/birthday-reminder.blade.php -->

<!DOCTYPE html>
<html>
<head>
    <title>Birthday Reminder</title>
</head>
<body>
    <h1>Hello {{ $user->name }},</h1>
    <p>Just a reminder that your birthday is tomorrow. Don't forget to celebrate!</p>
</body>
</html>
</pre></code>


Enfin, configurez la tâche planifiée (cron job) pour exécuter le contrôleur chaque jour. Ouvrez le fichier app/Console/Kernel.php et ajoutez la définition de la tâche planifiée dans la méthode schedule :

<code><pre>

// app/Console/Kernel.php

protected function schedule(Schedule $schedule)
{
    // Exécuter le contrôleur BirthdayReminderController tous les jours à minuit
    $schedule->call('\App\Http\Controllers\BirthdayReminderController@sendBirthdayReminders')
             ->dailyAt('00:00');
}

</pre></code>


Assurez-vous que la tâche planifiée est correctement configurée sur votre serveur pour qu'elle s'exécute quotidiennement à minuit.

Avec ces étapes, vous avez maintenant un contrôleur Laravel qui enverra un e-mail de rappel à un utilisateur un jour avant son anniversaire. Le contrôleur sera exécuté chaque jour grâce à la tâche planifiée, et les e-mails seront envoyés aux utilisateurs dont l'anniversaire est prévu pour le lendemain.

