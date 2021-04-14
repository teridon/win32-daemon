# NAME

Win32::Daemon - Extension enabling Win32 Perl scripts to run as a true Win32 service.

# SYNOPSIS

```perl
use Win32::Daemon;
Win32::Daemon::StartService();
# ...process Perl code...
Win32::Daemon::StopService();
```

# DESCRIPTION

This extension enables a Win32 Perl script to act as a true Win32 service.

# FUNCTIONS

## Function List

> `AcceptedControls()`
>
> `CallbackTimer()`
>
> `CreateService()`
>
> `ConfigureService()`
>
> `QueryServiceConfig()`
>
> `DeleteService()`
>
> `GetLastError()`
>
> `GetSecurity()`
>
> `GetServiceHandle()`
>
> `HideService()`
>
> `QueryLastMessage()`
>
> `RegisterCallbacks()`
>
> `RestoreService()`
>
> `SetSecurity()`
>
> `SetServiceBits()`
>
> `ShowService()`
>
> `StartService()`
>
> `State()`
>
> `StopService()`
>
> `Timeout()`

## Function Descriptions

- AcceptedControls( \[$NewControls\] )

    This function queries (and optionally sets) the current list of controls that the service registers for.
    By registering for a control the script is notifying the SCM that it is accepting the specified
    control messages. For example, if you specify the `SERVICE_ACCEPT_PAUSE_CONTINUE` control then
    the SCM knows that the script will accept and process any attempt to pause and continue (resume
    from paused state) the service.

    Recognized accepted controls:

    ```perl
    SERVICE_ACCEPT_STOP............The service accepts messages to stop.
    SERVICE_ACCEPT_PAUSE_CONTINUE..The service accepts messages to pause
                                   and continue.
    SERVICE_ACCEPT_SHUTDOWN........The service accepts messages to
                                   shutdown the system: when the OS is
                                   shutting down the service will be
                                   notified when it has accepted this
                                   control.
    ```

    Following controls are only recognized on Windows 2000 and higher:

    ```
    SERVICE_ACCEPT_PARAMCHANGE.....The service accepts messages
                                   notifying it of any parameter change
                                   made to the service.
    SERVICE_ACCEPT_NETBINDCHANGE...The service accepts messages
                                   notifying it of any network binding
                                   changes.
    ```

    By default all of these controls are accepted. To change this pass in a value consisting of
    any of these values OR'ed together.

    **NOTE:** You can query and set these controls at any time. However it is only supported to
    set them before you start the service (calling the `StartService()` function).

- CallbackTimer( \[ $NewTimerValue \] )

    This function returns the value of the callback timer. The value is in milliseconds.
    This value indicates how often the "Running" callback subroutine will be called. Note
    that the calling of this routine will be blocked by any other callback.

    If you pass in a value it will reset the timer to the specified frequency. Passing in
    a 0 will disable all "Running" callbacks. Passing in -1 will toggle the state between
    calling the "Running" callback subroutine and not calling it.

- CreateService ( \\%ServiceInfo )

    This function creates a new service in the system configuration. The
    return is TRUE if the service was created, and FALSE otherwise. If an error
    occurred, call GetLastError to retrieve the actual error code.

    **NOTE:** This function will fail if the script is not running with administrator
    privileges.

    The hash describes the service to be created. The keys are:

    - `name`

        The 'internal' service name; that is, the name of the
        registry key used to store the information on this service.

    - `display`

        The 'display' service name; that is, the name displayed
        by the services control panel or MMC plugin.

    - `path`

        The full path name to the executable. This should be the path to your Perl
        executable, which will normally be the contents of `$^X`.

        **NOTE:** If you are using a compiled perl script (such as one
        generated with PerlApp) as opposed to a text based perl script file then this
        value must point to the actual compiled script's executable (eg. `MyCompiledPerlService.exe`)
        instead of (`$^X` which usually points to `perl.exe`). You can specify
        any parameters to pass into the service using the `parameters` key.

    - `user`

        The username the service is to run under; this is optional.

    - `password`

        The password to be used to log in the service; this is
        technically optional, but needs to be specified if `user` is.

    - `parameters`

        The parameters to be passed to Perl; in other words, the command line you
        would execute interactively, but without the leading \`\`perl ''. The `parameters` key
        value is appended to the `path` key when starting the service.
        Typically this will be something like:

        ```
        MyPerlScript.pl /a /b /c
        ```

    - `machine`

        The name of the machine to create the service on. Omission
        or an empty string specify the machine executing the call.

    - `service_type`

        An integer representing the type of the service;
        defaults to `SERVICE_WIN32_OWN_PROCESS`.

    - `start_type`

        An integer specifying how (or whether) the service is
        to be started. The default is `SERVICE_AUTO_START`.

    - `error_control`

        An integer specifying how the Service Control Manager
        is to react if the service fails to start. The default is
        `SERVICE_ERROR_IGNORE`, which in fact gets you an error log entry.

    - `load_order`

        The name of the load order group of which this service
        is a member. The default is membership in no group. See value
        `ServiceGroupOrder` in registry key
        `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control`
        for the names.

    - `tag_id`

        An integer representing the startup order of the service
        within its load ordering group.

    - `dependencies`

        A reference to the _internal_ names of services and/or
        load ordering groups upon which this service depends. The default is
        no dependencies. Load order group names are prefixed with a '+' to
        distinguish them from service names.

    - `description`

        A short text description of the service, displayed
        (at least) as flyover help by the MMC "services" plugin.

- ConfigureService( \\%ServiceInfo )

    Modify a service created with `CreateService`. Same arguments as
    `CreateService`.

    If you specify a `parameters` key you MUST specify a `path` key.

- DeleteService ($Machine, $ServiceName )

    This function deletes an existing service. The return is TRUE if the
    service was deleted, and FALSE otherwise. If an error occurred, call
    GetLastError to retrieve the actual error code.

    The arguments are the name of the machine (an empty string specifies
    the machine executing the call), and the 'internal' service name (i.e.
    the string passed in the `name` element when the service was created).

    A running service may not be deleted.

- GetSecurity( $Machine, $ServiceName )

    This will return a binary Security Descriptor (SD) that is associated with the
    specified service on the specified machine.

    The SD is in self-relative format. It can be imported into a `[Win32::Perms](https://metacpan.org/pod/Win32::Perms)` object using
    the `Win32::Perms` object's `Import()` method.

- RegisterCallbacks( $CodeRef | \\%Hash )

    This will register specified code subroutines that will be called when specified
    events take place. For example if you register a subroutine with the
    `pause` event then this routine will be called when there is an attempt to pause the
    service. Not all events must have callbacks registered.

    If only a reference to a subroutine is passed in then it will be called for each and every
    event. You can pass in a hash containing particular key names (listed below) with
    code references.

    Possible hash key names:

    ```perl
    Key Name                 Event
    -------------            --------------------------------------
    start....................The service is starting.
    pause....................The service is entering a paused state.
    continue.................The service is resuming from a paused
                             state.
    stop.....................The service is stopping (see note below).
    running..................The service is running (see note below).
    interrogate..............The service is being queried for
                             information.
    shutdown.................The system is being shut down.
    preshutdown..............The system is about to begin shutting
                             down (Vista+ only).
    param_change.............There has been a parameter change to
                             the system.
    net_bind_add.............A new network binding has been made.
    net_bind_remove..........A network binding has been removed.
    net_bind_enable..........A network binding has been enabled.
    net_bind_disable.........A network binding has been disabled.
    device_event.............A device has generated some event.
    hardware_profile_change..A change has been made to the system's
                             hardware profile.
    power_event..............A power event has occured (eg change to
                             battery power).
    session_change...........There has been a change in session.
    user_defined.............A user defined event has been sent to
                             the service.
    ```

    **NOTES:**

    - The `Stop` state

        When a service calls into the registered "stop" callback routine
        the script should call the `StopService()` function. This tells the service to terminate
        and return back to the Perl script. This is the only way for the service to know that it
        must stop.

    - The `Running` state

        Periodically the extension will call into a registered
        "Running" subroutine. This allows the script to process data. This routine should be fast
        and return quickly otherwise it will block other callback events from being run. The
        frequency of calling the "Running" subroutine is dictated by the callback timer value
        passed into `StartService()` and any changes made to this value by calling into
        `CallbackTimer()`.

- SetSecurity( $Machine, $ServiceName, $BinarySD | $Win32PermsObject )

    This applies the specified Security Descriptor (SD) to the specified service on the
    specified machine. You must have appropriate permissions to call this function.

    The specified SD can be either a binary SD (in self-relative or absolute format) or
    it can be a Win32::Perms object.

    This only sets the DACL and SACL. The owner and group are not set even if they are
    specified in the SD.

- StartService( \[ \\%Context, $CallbackTimer \] )

    This starts a new service thread. The script should call this as soon as possible.  When
    the service manager starts the service Perl is started and the script is loaded.

    This function returns the thread handle of the service thread. If you call into this more
    than once it will only return the thread handle (it won't create another new service thread).

    - Callback Mode

        If the script has already registered callback routines (using `RegisterCallbacks()`) then
        the call into `StartService()` will not return until the service has stopped. However
        callbacks will be made for each state change and callback timer timeout (refer to `RegisterCallbacks()`).

- StopService()

    This will instruct the service to terminate.

- Timeout( \[$TimeoutValue\] )

    This function sets the new timeout value indicating how long a command will wait before
    Win32::Daemon tells the Service Control Manager that the command failed.

- QueryLastMessage( \[$fResetMessage\] )

    This function returns the last message that the service manager has sent to the service.

    Pass in a non zero value to reset the pending message to `SERVICE_CONTROL_NONE`. This way
    your script can tell when two of the same messages come in.

    Occasionally the service manager will send messages to the service. These messages
    typically request the service to change from one state to another.  It is important that
    the Perl script responds to each message otherwise the service manager becomes confused
    about the current state of the service. For example, if the service manager is submits
    a `SERVICE_PAUSE_PENDING` then it expects the Perl script to recognize the change to a paused
    state and submit the new state by calling `State( SERVICE_PAUSED )`.

    You can update the service manager with the current status using the `State()` function.

    Possible values returned are:

    ```perl
    Valid Service Control Messages
    ------------------------------
    SERVICE_CONTROL_NONE............No message is pending.
    SERVICE_CONTROL_STOP............The SCM is requesting the service to
                                    stop. This results in State()
                                    reporting SERVICE_STOP_PENDING.
    SERVICE_CONTROL_PAUSE...........The SCM is requesting the service to
                                    pause. This results in State()
                                    reporting SERVICE_PAUSE_PENDING.
    SERVICE_CONTROL_CONTINUE........The SCM is requesting the service to
                                    continue from a paused state. This
                                    results in State() reporting
                                    SERVICE_CONTINUE_PENDING.
    SERVICE_CONTROL_INTERROGATE.....The service manager is querying the
                                    service's state

    SERVICE_CONTROL_USER_DEFINED....This is a user defined control.
                                    There are 127 of these beginning
                                    with SERVICE_CONTROL_USER_DEFINED
                                    as the base.
    ```

    Windows 2000 specific messages:

    ```perl
    SERVICE_CONTROL_SHUTDOWN........The machine is shutting down. This
                                    indicates that the service has
                                    roughly 20 seconds to clean up and
                                    terminate. This time can be extended
                                    by submitting SERVICE_STOP_PENDING
                                    via the State() function.
    SERVICE_CONTROL_PARAMCHANGE.....Service parameters have been
                                    modified.
    SERVICE_CONTROL_NETBINDADD......A network binding as been added.
    SERVICE_CONTROL_NETBINDREMOVE...A network binding has been removed.
    SERVICE_CONTROL_NETBINDENABLE...A network binding has been enabled.
    SERVICE_CONTROL_NETBINDDISABLE..A network binding has been disabled.
    SERVICE_CONTROL_DEVICEEVENT.....A device has generated some event.
    SERVICE_CONTROL_HARDWAREPROFILECHANGE
                                    A change has been made to the
                                    system's hardware profile.
    SERVICE_CONTROL_POWEREVENT......A power event has occured (eg change
                                    to battery power).
    SERVICE_CONTROL_SESSIONCHANGE...There has been a change in session.
    ```

    Windows Vista+ specific messages:

    ```
    SERVICE_CONTROL_PRESHUTDOWN ....The machine is about to shut down.
                                    This provides the service much more
                                    time to shutdown than
                                    SERVICE_CONTROL_SHUTDOWN.
    ```

    **Note:** When the system shuts down it will send a `SERVICE_CONTROL_SHUTDOWN` message. The
    Perl script has approximately 20 seconds to perform any shutdown activities before the
    Control Manger stops the service. If more time is needed call the `State()` function
    passing in the `SERVICE_STOP_PENDING` control message along with how many seconds it will
    take to shutdown the service. This time value is only an estimate. When the service is
    finally ready to stop it must submit the `SERVICE_STOPPED` message as in:

    ```
    if( SERVICE_CONTROL_SHUTDOWN == State() )
    {
        Win32::Daemon::State( SERVICE_STOP_PENDING, 30_000 );
        #...process code...
        Win32::Daemon::State( SERVICE_STOPPED );
    }
    ```

- State(\[$NewState \[, $Hint \] || \\%Hash \] )

    This function returns the current state of the service.  It can optionally update the
    status of the service as well.  This is the last status reported to the service manager.

    Optionally you can pass in a value that will be sent to the service manager.
    Optionally you can pass in a numeric value indicating the "hint". This is the number of
    milliseconds the SCM can expect to wait before the service responds to the request. For example,
    if your service script reports a hint of 30,000 milliseconds means that the SCM will have to wait
    for 30 seconds for the script to change the service's state before deciding that the
    script is non responsive.

    If you are setting/updating the state instead of passing in the state and wait hint you could
    pass in a hash reference. This allows you to specify the state, wait hint and error state. You
    can use the following keys:

    ```
    Hash Key
    --------
    state..........Valid service state (see table below).
    waithint.......A wait hint explained above. This is in milliseconds.
    error..........Any 32 bit error code. This is what will be reported
                   if an application queries the error state of the
                   service. It is also what is reported if a call to
                   start the services fails.
                   To reset an error state pass in NO_ERROR.
                   The only invalid error value is 0xFFFFFFFF.
    ```

    Example of passing in an error:

    ```perl
    Win32::Daemon::State( { error => 0x12345678 } );
    # Later to reset the error:
    Win32::Daemon::State( { error => NO_ERROR } );
    ```

    Possible values returned (or submitted):

    ```perl
    Valid Service States
    --------------------
    SERVICE_NOT_READY..........The SCM has not yet been initialized. If
                               the SCM is slow or busy then this value
                               will result from a call to State().
                               If you get this value, just keep calling
                               State() until you get
                               SERVICE_START_PENDING.
    SERVICE_STOPPED............The service is stopped.
    SERVICE_RUNNING............The service is running.
    SERVICE_PAUSED.............The service is paused.
    SERVICE_START_PENDING......The service manager is attempting to
                               start the service.
    SERVICE_STOP_PENDING.......The service manager is attempting to
                               stop the service.
    SERVICE_CONTINUE_PENDING...The service manager is attempting to
                               resume the service.
    SERVICE_PAUSE_PENDING......The service manager is attempting to
                               pause the service.
    ```

# Callbacks

Callbacks were introduced in version v20030617.

The Win32::Daemon supports the concept of event callbacks. This allows a script to
register a particular subroutine with a particular event. When the event occurs it
will call the Perl subroutine registered with that event. This can make it very simple
to write scripts.

You register a callback subroutine by calling into the `RegisterCallbacks()` function.
You can pass in a code reference or a hash. A code reference will register the specified
subroutine with all events. A hash allows you to pick which events you want to
register for which subroutines. You do not have to register all events. If an event is
not registered for a subroutine then the script will not be notified when the event
occurs.

At a minimum a script should register for the 'Start' and 'Running' states. This enables
the script to actually start and to periodically process data.

When an event callback occurs the subroutine should change the state accordingly by
passing in the new state into `State()`. For example the 'Start' callback would call
`State( SERVICE_RUNNING )` to inform the service that it is officially running. Another
example is the 'Pause' state should call `State( SERVICE_PAUSED )` to inform the service
that it is officially paused.

Once callback subroutines are registered the script enters the service mode by calling
`StartService()`. This will being the process of calling the event callback routines.
Note that when callback routines are registered the `StartService()` function will not
return until a callback routine calls `StopService()` (typically the 'Stop' event callback
would call `StopService()`.

When calling into `StartService()` you can pass in a hash reference. This reference is known as
a "context" hash. For every callback the hash will be passed into the callback routine. This enables
a script to query and set data in the hash--essentially letting you pass information across to
different callback events. This context hash is not required.

When a callback is made it always passes two parameters in: $State and $Context. $State is simply
the state change that caused the callback. This represents the event that took place (e.g. `SERVICE_PAUSE_PENDING`,
`SERVICE_START_PENDING`, etc). The $Context is a reference to the context hash that was passed into
the `StartService()` function.

A typical callback routine should look similar to:

```perl
sub Callback_Start
{
    my( $Event, $Context ) = @_;
    $Context->{last_event} = $Event;

    # ...do some work here...

    # Tell the service manager that we have now
    # entered the running state.
    Win32::Daemon::State( SERVICE_RUNNING );
    return();
}
```

Refer to `Example 4: Using a single callback` and `Example 5: Using different callback routines` for an example of using callbacks.

# COMPILED PERL APPLICATIONS

Many users like to compile their perl scripts into executable programs. This way it is much easier to copy them around
from machine to machine since all necessary files, packages and binaries are compiled into one .exe file. These compiled
perl scripts are compatible with Win32::Deamon as long as you install it correctly.

If you are going to compile your Win32::Daemon based perl script into an .exe there is nothing unique you need to do
to your Win32::Daemon code with one single exception of the call into Win32::Daemon::`CreateService()`. When passing in
the 'path' and 'parameters' values into `CreateService()` observe the following simple rules:

- If using a Perl script

    ```perl
    path........The full path to the Perl interpeter ($^X).
                This is typically:
                  c:\perl\bin\perl.exe

    parameters..This value MUST start with the full path to the perl
                script file and append any parameters
                that you want passed into the service. For
                Example:
                  c:\scripts\myPerlService.pl -param1 -param2 "c:\\Param2Path"
    ```

- If using a compiled Perl application

    ```perl
    path........The full path to the compiled Perl application.
                For example:
                  c:\compiledscripts\myPerlService.exe

    parameters..This value is just the list of  parameters
                that you want passed into the service. For
                Example:
                  -param1 -param2 "c:\\Param2Path"
    ```

Refer to ["Example 3: Install the service"](#example-3-install-the-service) for an example.

# EXAMPLES

## Example 1: Simple Service

This example service will delete all .tmp files from the c:\\temp directory every
time it starts.  It will immediately terminate.

```perl
use Win32::Daemon;

# Tell the OS to start processing the service...
Win32::Daemon::StartService();

# Wait until the service manager is ready for us to continue...
while( SERVICE_START_PENDING != Win32::Daemon::State() )
{
    sleep( 1 );
}

# Now let the service manager know that we are running...
Win32::Daemon::State( SERVICE_RUNNING );

# Okay, go ahead and process stuff...
unlink( glob( "c:\\temp\\*.tmp" ) );

# Tell the OS that the service is terminating...
Win32::Daemon::StopService();
```

This particular example does not really illustrate the capabilities of a Perl based service.

## Example 2: Typical skeleton code

```perl
# This style of Win32::Daemon use is obsolete. It still works but the
# callback model is more efficient and easier to use. Refer to examples 4
#and 5.
use Win32;
use Win32::Daemon;
$SERVICE_SLEEP_TIME = 20; # 20 milliseconds
$PrevState = SERVICE_START_PENDING;
Win32::Daemon::StartService();
while( SERVICE_STOPPED != ( $State = Win32::Daemon::State() ) )
{
  if( SERVICE_START_PENDING == $State )
  {
      # Initialization code
      Win32::Daemon::State( SERVICE_RUNNING );
      $PrevState = SERVICE_RUNNING;
  }
  elsif( SERVICE_STOP_PENDING == $State )
  {
    Win32::Daemon::State( SERVICE_STOPPED );
  }
  elsif( SERVICE_PAUSE_PENDING == $State )
  {
    # "Pausing...";
    Win32::Daemon::State( SERVICE_PAUSED );
    $PrevState = SERVICE_PAUSED;
    next;
  }
  elsif( SERVICE_CONTINUE_PENDING == $State )
  {
    # "Resuming...";
    Win32::Daemon::State( SERVICE_RUNNING );
    $PrevState = SERVICE_RUNNING;
    next;
  }
  elsif( SERVICE_STOP_PENDING == $State )
  {
    # "Stopping...";
    Win32::Daemon::State( SERVICE_STOPPED );
    $PrevState = SERVICE_STOPPED;
    next;
  }
  elsif( SERVICE_RUNNING == $State )
  {
    # The service is running as normal...
    # ...add the main code here...

  }
  else
  {
    # Got an unhandled control message. Set the state to
    # whatever the previous state was.
    Win32::Daemon::State( $PrevState );
  }

  # Check for any outstanding commands. Pass in a non zero value
  # and it resets the Last Message to SERVICE_CONTROL_NONE.
  if( SERVICE_CONTROL_NONE != ( my $Message = Win32::Daemon::QueryLastMessage( 1 ) ) )
  {
    if( SERVICE_CONTROL_INTERROGATE == $Message )
    {
      # Got here if the Service Control Manager is requesting
      # the current state of the service. This can happen for
      # a variety of reasons. Report the last state we set.
      Win32::Daemon::State( $PrevState );
    }
    elsif( SERVICE_CONTROL_SHUTDOWN == $Message )
    {
      # Yikes! The system is shutting down. We had better clean up
      # and stop.
      # Tell the SCM that we are preparing to shutdown and that we
      # expect it to take 25 seconds (so don't terminate us for at
      # least 25 seconds)...
      Win32::Daemon::State( SERVICE_STOP_PENDING, 25000 );
    }
  }
  # Snooze for awhile so we don't suck up cpu time...
  Win32::Sleep( $SERVICE_SLEEP_TIME );
}
# We are done so close down...
Win32::Daemon::StopService();
```

## Example 3: Install the service

For the 'path' key the $^X equates to the full path of the
perl executable.
Since no user is specified it defaults to the LocalSystem.

```perl
use Win32::Daemon;
# If using a compiled perl script (eg. myPerlService.exe) then
# $ServicePath must be the path to the .exe as in:
#    $ServicePath = 'c:\CompiledPerlScripts\myPerlService.exe';
# Otherwise it must point to the Perl interpreter (perl.exe) which
# is conviently provided by the $^X variable...
my $ServicePath = $^X;

# If using a compiled perl script then $ServiceParams
# must be the parameters to pass into your Perl service as in:
#    $ServiceParams = '-param1 -param2 "c:\\Param2Path"';
# OTHERWISE
# it MUST point to the perl script file that is the service such as:
my $ServiceParams = 'c:\perl\scripts\myPerlService.pl -param1 -param2 "c:\\Param2Path"';


my %service_info = (
    machine =>  '',
    name    =>  'PerlTest',
    display =>  'Oh my GOD, Perl is a service!',
    path    =>  $ServicePath,
    user    =>  '',
    pwd     =>  '',
    description => 'Some text description of this service',
    parameters => $ServiceParams
);
if( Win32::Daemon::CreateService( \%service_info ) )
{
    print "Successfully added.\n";
}
else
{
    print "Failed to add service: " . Win32::FormatMessage( Win32::Daemon::GetLastError() ) . "\n";
}
```

## Example 4: Using a single callback

In this example only one subroutine is used for all callbacks. The CallbackRoutine()
subroutine will receive all event callbacks. Basically this callback routine will
have to do essentially the same thing that the main while loop in
["Example 2: Typical skeleton code"](#example-2-typical-skeleton-code) does.

```perl
use Win32::Daemon;
Win32::Daemon::RegisterCallbacks( \&CallbackRoutine );
%Context = (
    count   =>  0,
    start_time => time(),
);
# Start the service passing in a context and
# indicating to callback using the "Running" event
# every 2000 milliseconds (2 seconds).
Win32::Daemon::StartService( \%Context, 2000 );

sub CallbackRoutine
{
    my( $Event, $Context ) = @_;

    $Context->{last_event} = $Event;
    if( SERVICE_RUNNING == $Event )
    {
        # ... process your main stuff here...
        # ... note that here there is no need to
        #     change the state

    }
    elsif( SERVICE_START_PENDING == $Event )
    {
        # Initialization code
        # ...do whatever you need to do to start...

        $Context->{last_state} = SERVICE_RUNNING;
        Win32::Daemon::State( SERVICE_RUNNING );
    }
    elsif( SERVICE_PAUSE_PENDING == $Event )
    {
        $Context->{last_state} = SERVICE_PAUSED;
        Win32::Daemon::State( SERVICE_PAUSED );

    }
    elsif( SERVICE_CONTINUE_PENDING == $Event )
    {
        $Context->{last_state} = SERVICE_RUNNING;
        Win32::Daemon::State( SERVICE_RUNNING );
    }
    elsif( SERVICE_STOP_PENDING == $Event )
    {
        $Context->{last_state} = SERVICE_STOPPED;
        Win32::Daemon::State( SERVICE_STOPPED );

        # We need to notify the Daemon that we want to stop callbacks
        # and the service.
        Win32::Daemon::StopService();
    }
    else
    {
        # Take care of unhandled states by setting the State()
        # to whatever the last state was we set...
        Win32::Daemon::State( $Context->{last_state} );
    }
    return();
}
```

## Example 5: Using different callback routines

```perl
use Win32::Daemon;
Win32::Daemon::RegisterCallbacks( {
    start       =>  \&Callback_Start,
    running     =>  \&Callback_Running,
    stop        =>  \&Callback_Stop,
    pause       =>  \&Callback_Pause,
    continue    =>  \&Callback_Continue,
} );

my %Context = (
    last_state => SERVICE_STOPPED,
    start_time => time(),
);

# Start the service passing in a context and
# indicating to callback using the "Running" event
# every 2000 milliseconds (2 seconds).
Win32::Daemon::StartService( \%Context, 2000 );

sub Callback_Running
{
    my( $Event, $Context ) = @_;

    # Note that here you want to check that the state
    # is indeed SERVICE_RUNNING. Even though the Running
    # callback is called it could have done so before
    # calling the "Start" callback.
    if( SERVICE_RUNNING == Win32::Daemon::State() )
    {
        # ... process your main stuff here...
        # ... note that here there is no need to
        #     change the state
    }
}

sub Callback_Start
{
    my( $Event, $Context ) = @_;
    # Initialization code
    # ...do whatever you need to do to start...

    $Context->{last_state} = SERVICE_RUNNING;
    Win32::Daemon::State( SERVICE_RUNNING );
}

sub Callback_Pause
{
    my( $Event, $Context ) = @_;
    $Context->{last_state} = SERVICE_PAUSED;
    Win32::Daemon::State( SERVICE_PAUSED );
}

sub Callback_Continue
{
    my( $Event, $Context ) = @_;
    $Context->{last_state} = SERVICE_RUNNING;
    Win32::Daemon::State( SERVICE_RUNNING );
}

sub Callback_Stop
{
    my( $Event, $Context ) = @_;
    $Context->{last_state} = SERVICE_STOPPED;
    Win32::Daemon::State( SERVICE_STOPPED );

    # We need to notify the Daemon that we want to stop callbacks and the service.
    Win32::Daemon::StopService();
}
```

# NOTES

## Timer/Running Callbacks:

Starting with build 20080321 the "running" callback is deprecated and replaced with the
"timer" callback. Scripts should no longer test for a state of SERVICE\_RUNNING but instead check
for the state of SERVICE\_CONTROL\_TIMER to indicate whether or not a callback has occurred
due to a timer.

If a script...

- ...registers for the "running" callback it will continue to work
as expected: timer expiration results in a callback to the subroutine registered for the "running"
callback passing in a value of SERVICE\_RUNNING.
- ...registers for the "timer" callback then timer expiration results in a callback to the
subroutine registered for the "timer" callback, passing in a value of SERVICE\_CONTROL\_TIMER.
- ...registers for both "running" and "timer" then only Win32::Daemon treats it as if only
"timer" was registered (see above for behavior).
- ...registers for everything by passing one subroutine reference into Win32::Daemon::Callback()
then both "running" and "timer" are registered and only "timer" is recognized (see previous
2 behaviors above).

Legacy scripts which call Win32::Daemon::Callback() passing in only one catchall subroutine reference
will be most impacted as they will expect.

# SEE ALSO

[MSDN: _Service Control Manager_](http://msdn.microsoft.com/fr-fr/library/ms685150)

[MSDN: _Service Functions_](http://msdn.microsoft.com/fr-fr/library/ms685942%28v=VS.85%29.aspx)

# AUTHOR

Dave Roth, Roth Consulting, http://www.roth.net/

# CONTRIBUTORS

Haiko Strotbek <haiko@strotbek.com>

Jan Dubois <jand@activestate.com>

Marc Pijnappels <marc.pijnappels@nec-computers.com>

Olivier Mengué <dolmen@cpan.org>

# SUPPORT

Dave has retired from active development of this module.  It is now
being maintained as part of the libwin32 project <libwin32@perl.org>.

# COPYRIGHT

Copyright © 1998 - 2011 the Win32::Daemon ["AUTHOR"](#author) and ["CONTRIBUTORS"](#contributors)
as listed above.

# LICENSE

This library is free software and may be distributed under the same terms
as perl itself.
