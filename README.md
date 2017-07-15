# tclogger

Simplified logging for node.js modules.

## Features

* any code running in the `node` instance, including external modules, can append log messages.
* the external modules don't need any configuration knowledge to send messages to the log.
* there is no need to pass around a `syslog` object to every function that needs to log something.
* log messages can be directed anywhere, not just to syslog and console.
* log messages can go to 0, 1, or many destinations (/dev/null, syslog, file, rabbitmq, e-mail, XMPP, etc).
* a log destination can be turned on or off at runtime.
* logged objects are automatically formatted into key=value strings (great for sending messages to [splunk](http://www.splunk.com/)).
* certain fields can be censored to avoid accidentally logging sensitive information.
* formatted log messages are returned by tclogger to the caller.
* it accepts multiple arguments and printf-style formats just like `console.log`.
* defaults can be supplied that are included in every message.

## Theory of Operation

The module provides log functions and the arguments work just like [console.log()](https://nodejs.org/api/console.html#console_console_log_data),
supporting a variable number of arguments plus formatting.

When invoked, the logger will format the log message using [tclogformat](https://github.com/tcort/tclogformat)
(for example, a JSON object like `[ { name: 'Tom' }, { name: 'Phil' } ]` becomes `0.name=Tom 1.name=Phil`).
The log level and message are emitted as a `log` event though the `process` event emitter. The main
application will provide an event listener to forward the log message to syslog or any other destination
(RabbitMQ, log file, database, etc). Finally, the logging function returns the formatted log message
which can be displayed/returned to the user if desired.

## Transports

Log messages are emitted as `log` events. Event listeners should be installed to receive the events and send them over
the appropriate transport.

To find the available transports, just search for `tclogger` and `transport`.

## Censorship

Any number of fields may be censored. This is useful when logging request objects to avoid accidentally logging
a credit card number, password, or other sensitive information.

## API

### log.inTestEnv(...)

Emits a `DEBUG` level log message.

### log.inProdEnv(...)

Emits an `INFO` level log message.

### log.toInvestigateTomorrow(...)

Emits a `WARN` level log message.

### log.wakeMeInTheMiddleOfTheNight(...)

Emits an `ERROR` level log message.

### log.defaults(...)

Returns a new curried `log()` function with baked in parameters that are included in all log messages.

Example:

    var mylog = log.defaults({ request_id: '7423927D-6F4E-43FE-846E-C474EA3488A3' }, 'foobar');

    mylog.inProdEnv('I love golf!');

    // emits --> { level: 'INFO', message: 'I love golf! request_id=7423927D-6F4E-43FE-846E-C474EA3488A3 foobar' }


### log.censor(arr)

Sets the list of fields to censor from all log messages. The parameter `arr` is an array which may contain any combination of strings and regular expression objects. The strings and regular expressions are used to match against the log message. To turn off censorship, call this function with an empty array `[]`.

Example:

    // set the list
    log.censor([ 'card_number', /pass(word)?/ ]);

    log.inProdEnv('first_name=John last_name=Doe card_number=1234123412341234 password=pizza');
    log.inProdEnv('first_name=%s last_name=%s card_number=%s password=%s', first_name_var, last_name_var, card_number_var, password_var);
    log.inProdEnv({ first_name: 'John', last_name: 'Doe', card_number: '1234123412341234', password: 'pizza' });

    // each one above emits the same thing -->
    // { level: 'INFO', message: 'first_name=John last_name=Doe card_number=[redacted] password=[redacted]' }

### log.censor()

Returns a list of fields that are presently being censored from all log messages.

Example:

    // get the list of censored fields
    console.log(log.censor());
    // prints --> [ 'card_number', /pass(word)?/ ]

## Testing

There is an automated test suite:

    npm test

## License

See [LICENSE.md](https://github.com/tcort/tclogger/blob/master/LICENCE.md).
