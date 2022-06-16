# Realtime base use

## configure backend

- step 1 create and account or use one create in pusher.com
- step 2 configure laravel (backend) with a package to connect to pusher
  - step 2.1 `composer require pusher/pusher-php-serve`
  - step 2.2 enabled broadcast provider in `config/app.php`
  - step 2.3 add in `.env` the values of environment variables to connect pusher
## configure frontend
- step 3 configure libraries from communicate frontend with backend
  - step 3.1 add libraries
    ```shell
    npm install --save-dev laravel-echo pusher-js
    ```
  - step 3.2 in file `resources/js/bootstrap.js` uncomment the next reference
    ```javascript
    import Echo from 'laravel-echo';
    window.Pusher = require('pusher-js');
    window.Echo = new Echo({
    broadcaster: 'pusher',
    key: process.env.MIX_PUSHER_APP_KEY,
    cluster: process.env.MIX_PUSHER_APP_CLUSTER,
    forceTLS: true
    });
    ```
  - step 3.3 execute in console
      ```shell
      npm run dev
    ```
    
## Create a basic notification 

### case: when login or logout a user 
run a notification when un user start or finish the session

- configure a visual element in the layouts for show the alert this step 
configures in  the main layout of our project specific add in the `resources/views/layouts/app.blade` inside the main tag this code: 
```html
    <main class="py-4">
        <div id="notification" class="alert mx-3 invisible"> example test</div>
        @yield('content')
    </main>
```
- add components of type Event in the backend use this command `php artisan make:event {event component name}` 
  if the command run the first time automatically created a new folder with name `Events` in the app with an archive with name pass in the command
  in this file we have the next code
```injectablephp
    class UserSessionChange implements ShouldBroadcast
    {
        use Dispatchable, InteractsWithSockets, SerializesModels;
    
        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }
    
        /**
         * Get the channels the event should broadcast on.
         *
         * @return \Illuminate\Broadcasting\Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('channel-name');
        }
    }
```


 - Create two listeners, one to listen login events and other to listen logout events 
```bash
 artisan make:listener BroadcastLoginNotification
 artisan make:listener BroadcastLogoutNotification
```


 - register a new listeners in the file locate in `app/Providers/EventServiceProvider.php`
```injectablephp
protected $listen = [
        Registered::class => [
            SendEmailVerificationNotification::class,
        ],
        Login::class => [
            BroadcastLoginNotification::class,
        ],
        Logout::class =>[
            BroadcastLogoutNotification::class,
        ]
    ];
```
- to check is the notification is trigger in the file `resources/js/app.js` add the code to show
visual notification when the events register are trigger.

``` javascript
Echo.channel('notifications')
.listen('UserSessionChange',(e)=>{
    const notificationElement = document.getElementById('notification');
    notificationElement.innerText = e.message;
    notificationElement.classList.remove('invisible');
    notificationElement.classList.remove('success');
    notificationElement.classList.remove('danger');
    notificationElement.classList.add('alert-'+ e.type);
});
```

### Broadcast in private channel 

- change the type of channel on the method `broadCastOn` into the file `app/Events/UserSessionChange.php`
```injectablephp
public function broadcastOn()
    {
        return new PrivateChannel('notifications');
    }
````

- for the frontend side change the call in the file `resources/js/app.js`
```js
Echo.private('notifications')
.listen('UserSessionChange',(e)=>{
})
```

- and in the file `routes/channels.php` add a new route
``` injectablephp 
Broadcast::channel('notifications', function ($user) {
return $user != null;
});
```

