---
title: Installation & setup
weight: 4
---

You can install the package via composer:

```bash
composer require spatie/laravel-health
```

## Publishing the config file

Optionally, you can publish the `health` config file with this command.

```bash
php artisan vendor:publish --tag="health-config"
```

This is the content of the published config file:

```php
return [

    /*
     * A result store is responsible for saving the results of the checks. The
     * `EloquentHealthResultStore` will save results in the database. You
     * can use multiple stores at the same time.
     */
    'result_stores' => [
        Spatie\Health\ResultStores\EloquentHealthResultStore::class,

        /*
        Spatie\Health\ResultStores\EloquentHealthResultStore\JsonFileHealthResultStore::class => [
            'disk' => 's3',
            'path' => 'health.json',
        ],

        */
    ],


    /*
     * The amount of days the `EloquentHealthResultStore` will keep history
     * before pruning items.
     */
    'keep_history_for_days' => 5,

    /*
         * You can get notified when specific events occur. Out of the box you can use 'mail' and 'slack'.
         * For Slack you need to install laravel/slack-notification-channel.
         *
         * You can also use your own notification classes, just make sure the class is named after one of
         * the `Spatie\Backup\Notifications\Notifications` classes.
         */
    'notifications' => [

        'notifications' => [
            Spatie\Health\Notifications\CheckFailedNotification::class => ['mail'],
        ],

        /*
         * Here you can specify the notifiable to which the notifications should be sent. The default
         * notifiable will use the variables specified in this config file.
         */
        'notifiable' => Spatie\Health\Notifications\Notifiable::class,

        /*
         * When checks start failing, you could potentially end up getting
         * a notification every minute. 
         * 
         * With this setting, notifications are throttled. By default, you'll
         * only get one notification per hour.
         */
        'throttle_notifications_for_minutes' => 60,

        'mail' => [
            'to' => 'your@example.com',

            'from' => [
                'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
                'name' => env('MAIL_FROM_NAME', 'Example'),
            ],
        ],

        'slack' => [
            'webhook_url' => '',

            /*
             * If this is set to null the default channel of the webhook will be used.
             */
            'channel' => null,

            'username' => null,

            'icon' => null,
        ],
    ],
];
```

## Migrating the database

When using the `EloquentHealthResultStore` the check results will be stored in the database.

To create the `check_result_history_items` table, you must create and run the migration.

```bash
php artisan vendor:publish --tag="health-migrations"
php artisan migrate
```

These steps are not necessary when using the `JsonFileResultStore`.

## Scheduling the commands

The checks will be triggered by executing a command. You should schedule the `RunHealthChecksCommand` to run every minute.

```php
// in app/Console/Kernel.php

protected function schedule(Schedule $schedule)
{
    $schedule->command(\Spatie\Health\Commands\RunHealthChecksCommand::class)->everyMinute();
}
```
