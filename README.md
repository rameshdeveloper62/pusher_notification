(1) first of all we should create table for store notification
	php artisan notifications:table
	php artisan migrate

(2) we create create sendMessage notification class
	App\Notifications\SendMessage.php
	-> php artisan make:notification SendMessage
	-> we used datatable type of notification 

		public function via($notifiable)
	    {
	        return ['database'];
	    }
	-> toArray() method for store data in notification table
	    public function toArray($notifiable)
	    {
	        return $this->message;
	    }

(3) we used by default user reference table, Notifiable trait is already defined in user model 
(4) we send notification when user add new message
	
	$to_user->notify(new SendMessage($notification));

(5) we can display current login user notification by $user->notifications.
	if we want to display only read,unread,all but we can able to display all type of notification

	-> read and unread = $user->notifications
	-> unread = $user->unreadNotifications
	-> Marking Notifications As Read

		$user = App\User::find(1);

		foreach ($user->unreadNotifications as $notification) {
		    $notification->markAsRead();
		}
		// all notification read by current user object
		$user->unreadNotifications->markAsRead();
		
		// mark as read by update method
		$user->unreadNotifications()->update(['read_at' => now()]);

	-> delete notification
		$user->notifications()->delete();
(6) we can change in config/app.php file for enable brodcast service provider
	
	uncomment this line :  App\Providers\BroadcastServiceProvider::class,

(7) change of BROADCAST_DRIVER=log to pusher in .env file
	
	BROADCAST_DRIVER=pusher

(8) add pusher key,secret,cluster in .env file

(9) create event for send notification 
	if you didn't add event name and listener so these command are run. it will automatically add in app/Providers/EventServiceprovider.php.

	php artisan make:event MessageEvent

	define channel name

	public function broadcastOn()
    {
        return new PrivateChannel("message-notification.{$this->data['to_user']}");
    }

(10) when user send message to other user.
	
	HomeController@store method
	event(new MessageEvent($notification));

(12) setting of pusher and echo in resources\js\boostrap.js file
	uncomment below code in boostrap file
	
	import Echo from 'laravel-echo'

	window.Pusher = require('pusher-js');

	window.Echo = new Echo({
    	broadcaster: 'pusher',
    	key: process.env.MIX_PUSHER_APP_KEY,
    	cluster: process.env.MIX_PUSHER_APP_CLUSTER,
    	encrypted: true
	});
	
(13)


