# lucid v1.4.0 <!-- omit in toc -->

**lucid** is an openHAB jsr223 Jython helper library. It's a derivative work based on [Steve Bate](https://github.com/steve-bate)'s great project [openHab2-jython](https://github.com/OH-Jython-Scripters/openhab2-jython).

## Deprecated project
This project is no longer supported, please consider using [openhab2-jython](https://github.com/OH-Jython-Scripters/openhab2-jython) instead.

**lucid** takes openHAB Jython scripting to a higher level and can be used for general scripting purposes, including defining rules, triggers and conditions. **lucid** aims to be easy to use and to provide a good documentation. (The documentation work has just begun)
- [Getting Started with lucid](#getting-started-with-lucid)
    - [Prerequisites](#prerequisites)
    - [Installing](#installing)
    - [Set up logging](#set-up-logging)
    - [Test your lucid installation](#test-your-lucid-installation)
    - [Configuration file](#configuration-file)
    - [Example scripts](#example-scripts)
- [Defining rules using rule scripts](#defining-rules-using-rule-scripts)
    - [Imports](#imports)
    - [Triggers](#triggers)
        - [Event-based Triggers](#event-based-triggers)
        - [Time-based (cron) Triggers](#time-based-cron-triggers)
        - [System-based Triggers](#system-based-triggers)
        - [Thing-based Triggers](#thing-based-triggers)
        - [Channel-based Triggers](#channel-based-triggers)
    - [Events](#events)
- [Loading and reloading the Jython rule binding](#loading-and-reloading-the-jython-rule-binding)
- [Contributing](#contributing)
    - [Native english speakers](#native-english-speakers)
    - [openHAB Jython Scripting on Slack](#openhab-jython-scripting-on-slack)
- [License](#license)
- [Acknowledgments](#acknowledgments)
- [Disclaimer](#disclaimer)

## Getting Started with lucid

These instructions will get you **lucid** up and running on your openHAB server.

### Prerequisites

- [openHAB](https://docs.openhab.org/index.html) version **2.3** or later
- The [Experimental Next-Gen Rule Engine](https://www.openhab.org/docs/configuration/rules-ng.html) add-on must be installed in openHAB 2.
- openHAB should be configured to use a [persistence service](https://www.openhab.org/docs/configuration/persistence.html).

### Installing

* Follow the documentation to install [JSR223 Jython Scripting](https://www.openhab.org/docs/configuration/jsr223-jython.html) unless you already done that. Now, according to the documentation you have already set the `EXTRA_JAVA_OPTS` environment variable in `/etc/default/openhab2` to something similar to this example:

   ```
  EXTRA_JAVA_OPTS=-Xbootclasspath/a:/home/pi/jython2.7.0/jython.jar \
  -Dpython.home=/home/pi/jython2.7.0 \
  -Dpython.path=/etc/openhab2/automation/lib/python
   ```
   The last line in the example above defines the path where openHAB can find your jython library files. You can choose a different directory for your installation. We prefer using the directory `/etc/openhab2/automation/lib/python` (The [JSR223 Jython Scripting](https://www.openhab.org/docs/configuration/jsr223-jython.html) documentation uses a different lib directory which is also OK) We will now simply refer to that directory as the **LIB-DIR**. It's the directory where openHAB will look for your jython library files.

* Download the [lucid archive file](https://github.com/OH-Jython-Scripters/lucid/archive/master.zip), extract it in a temporary location.
* From the extracted zip file, transfer the [lucid](https://github.com/OH-Jython-Scripters/lucid/tree/master/automation/lib/python/lucid) folder together with all its content (found in the zip file's automation/lib/python folder) into your LIB-DIR. 
* Change the owner, group and file permissions. E.g. cd into the LIB-DIR and run `sudo chown -R openhab:openhab lucid` followed by `sudo chmod -R 664 lucid`
* From the extracted zip file, transfer all the the files from the [jsr223](https://github.com/OH-Jython-Scripters/lucid/tree/master/automation/jsr223) (found in the zip file's automation/jsr223 folder) into your `automation/jsr223` directory
* Change the owner, group and file permissions. E.g. cd into your `automation/jsr223` directory and run `sudo chown openhab:openhab 000_*.py` followed by `sudo chmod 664 000_*.py`

* Create an openHAB item named `ZZZ_Test_Reload_Finished` and put it last in your items file. Make that item persisted "on change". In the example below, persistance "on change" is assigned to the group `G_PersistOnChange`
```
String ZZZ_Test_Reload_Finished (G_PersistOnChange) // Used for checking if reloading 
```

### Set up logging
You'd probably want to configure logging for lucid in the config file for logging. The config file for logging is org.ops4j.pax.logging.cfg located in the userdata/etc folder (manual setup) or in /var/lib/openhab2/etc (apt/deb-based setup). See the [documentation](https://www.openhab.org/docs/administration/logging.html#config-file). In the `OSGi appender` section, after line `log4j2.logger.org_eclipse_smarthome_automation.name = org.eclipse.smarthome.automation`, add
   ```
   log4j2.logger.lucid.level = DEBUG
   log4j2.logger.lucid.name = lucid
   ```

### Test your lucid installation
Put the [helloWorld.py](https://raw.githubusercontent.com/OH-Jython-Scripters/lucid/master/Script%20Examples/helloWorld.py) file from the examples in your `automation/jsr223` directory and watch the openHAB log file carefully. It should output "Hello world from lucid!" once every minute. Delete the `helloWorld.py` file when you are done.

### Configuration file
For some functionality, like the ability to send autoremote messages for example, there is some configuration to do. In LIB-DIR/lucid, rename the file example_config.py to config.py and edit the file to suit your needs. It should be quite self explanatory what it's all about. The configuration file can also be used to store custom config entries for all your jython scripts. The configuration entries are
available in the lucid rules that you define. 
```python
if (self.config.somerandomdata['anumber'] == 0):
    # Do something
```

Some script packages based on **lucid** may assume that you have defined a configuration file so it's recommended that you create one now.

### Example scripts
You are now ready to make your own scripts using lucid. Have a look at the [examples](https://github.com/OH-Jython-Scripters/lucid/tree/master/Script%20Examples) or continue at [Writing scripts](#writing-scripts).

## Defining rules using rule scripts
In order for your jython scripts to work with lucid, they need to
* Import mandatory libraries
* Define one or more rule classes with functions that return what triggers you wish to use and an execute function which is where you define what should be executed when the triggers fire.
* Add the rule class to the automation manager.

For example:
```python
from lucid.rules import rule, addRule
from lucid.triggers import ItemStateChangeTrigger

@rule
class ExampleRule(object):
    def getEventTriggers(self):
        return [
            ItemStateChangeTrigger('My_TestSwitch_1'),
        ]

    def execute(self, modules, inputs):
        self.log.info('Something has happened')

addRule(ExampleRule())
```
### Imports
Some useful text will soon be found here.

### Triggers
Before a rule scripts does anything, it has to be triggered. What kind of triggers that are available is very well documented for the rules DSL (using Xtend programming language) that you see [here](https://www.openhab.org/docs/configuration/rules-dsl.html#rule-triggers). (We have copied some text from that documentation and made some changes to it to suit **lucid**)

There are different categories of rule triggers:

1. Item(-Event)-based triggers: They react on events on the openHAB event bus, i.e. commands and status updates for items
2. Time-based triggers: They react at special times, e.g. at midnight, every hour, etc.
3. System-based triggers: They react on certain system statuses.
4. Thing-based triggers: They react on thing status, i.e. change from ONLINE to OFFLINE.
5. Channel-based triggers: They react on channels provided by some add-ons.

Here are the details for each category:

#### Event-based Triggers
You can listen to commands for a specific item, on status updates or on status changes (an update might leave the status unchanged). You can decide whether you want to catch only a specific command/status or any. Here is the syntax for all these cases (parts in square brackets are optional):

```python
ItemCommandTrigger('<item>', ['<command>']), # Item received command
ItemStateUpdateTrigger('<item>', ['<state>']), # Item received update
ItemStateChangeTrigger('<item>', ['<state>']), # Item changed
```

A simplistic explanation of the differences between command and update can be found in the article about [openHAB core actions](https://www.openhab.org/docs/configuration/actions.html#core-actions).

#### Time-based (cron) Triggers
**lucid** supports [cron expressions](http://www.quartz-scheduler.org/documentation/quartz-2.1.x/tutorials/tutorial-lesson-06).

```python
CronTrigger(EVERY_MINUTE), # Using one of the predefined cron expression strings
CronTrigger('3 11 09 * * ?'), # Runs at 09:11:03 every day
CronTrigger(EVERY_15_MINUTES), # Using one of the predefined cron expression strings
```

A cron expression takes the form of six or optionally seven fields:

* Seconds
* Minutes
* Hours
* Day-of-Month
* Month
* Day-of-Week
* Year (optional field)
For more information see the [Quartz documentation](http://www.quartz-scheduler.org/documentation/quartz-2.1.x/tutorials/tutorial-lesson-06).

You may also use [CronMaker](http://www.cronmaker.com/) or the generator at [FreeFormatter.com](http://www.freeformatter.com/cron-expression-generator-quartz.html) to generate cron expressions.

For your convenience there is a set of predefined cron expression constants available within the scope of the getEventTriggers function:

* `EVERY_10_SECONDS`
* `EVERY_15_SECONDS`
* `EVERY_30_SECONDS`
* `EVERY_MINUTE`
* `EVERY_OTHER_MINUTE`
* `EVERY_5_MINUTES`
* `EVERY_10_MINUTES`
* `EVERY_15_MINUTES`
* `EVERY_30_MINUTES`
* `EVERY_HOUR`
* `EVERY_6_HOURS`
* `EVERY_DAY_AROUND_NOON` Note that this will trigger at a every day "around noon" between 11:03 AM and 12:57 AM.

**NOTE!!** To avoid the "top of the minute problem" we space out our cronjobs (only those that are using predefined cron expressions) not to occur exactly on the minute or at the top of the hour. That will help your own server as well as shared resources that you'll be using to spread out the work more evenly. Each of the above predefined cron expressions will be randomized at openHAB reload. The `EVERY_HOUR` expression might for example be randomized to occur at *second :19, minute 32 every hour* It will still run once an hour but using the expressons above you can not be certain exactly when. If you for some reason would like to run a cron expression hourly, triggering at minute 0 and second 0 you should not use the predfined cron expressions.

You don't need to specifically import these cron expression string constants. Just use them like:

```python
CronTrigger(EVERY_HOUR), # Runs every hour but not on the minute 00
CronTrigger(EVERY_DAY_AROUND_NOON), # Runs every day around noon +- 1 hour
```

#### System-based Triggers
A single system-based trigger is provided by **lucid**.

* System started. System started is triggered upon openHAB startup, after the rule file containing the System started trigger is modified, or after item(s) related to that rule file are modified in a .items file.

* System shuts down isn't yet implemented.

Example:

```python
StartupTrigger(),
```

#### Thing-based Triggers
**lucid** doesn't currently support thing based triggers.

#### Channel-based Triggers
Some add-ons provide trigger channels. Compared with other types of channels, a trigger channel provides information about discrete events, but does not provide continuous state information.

Your rules can take actions based upon trigger events generated by these trigger channels. You can decide whether you want to catch only a specific or any trigger the channel provides. Here is the syntax for these cases (parts in square brackets are optional):

```python
ChannelEventTrigger('<triggerChannel>','<triggerEvent>')
```

triggerChannel is the identifier for a specific channel.

When a binding provides such channels, you can find the needed information in the corresponding binding documentation. There is no generic list of possible values for triggerEvent, The triggerEvent(s) available depend upon the specific implementation details of the binding.

Example:
```python
ChannelEventTrigger('astro:sun:local:rise#event','START'),
ChannelEventTrigger('astro:sun:local:set#event','START'),
ChannelEventTrigger('astro:sun:local:civilDusk#event','START'),
ChannelEventTrigger('astro:sun:local:civilDusk#event','END'),
```

### Events
Some useful text will soon be found here.

## Loading and reloading the Jython rule binding
To restart the Jython binding and reload all the Jython libs & scripts, [Access the console](https://www.openhab.org/docs/administration/console.html) and isse the following command:
```
bundle:restart org.eclipse.smarthome.automation.module.script.rulesupport
```

To make sure that the Jython rule binding binding is loaded as after all the items are initialized, (This has to be done after every OpenHAB2 update) [Access the console](https://www.openhab.org/docs/administration/console.html) and isse the following command:
```
bundle:start-level org.eclipse.smarthome.automation.module.script.rulesupport 90
```

## Contributing
There are several ways to contribute to this project.

### Native english speakers
We'd need some help from native english speakers to correct and improve this documentation regarding the language.

### openHAB Jython Scripting on Slack
OH-Jython-Scripters now has a Slack channel! It will help us to make sense of our work, and drive our efforts in Jython scripting forward. So if you are just curious, got questions, need support or just like to hang around, don't hesitate, join [**openHAB Jython Scripting on Slack**](https://join.slack.com/t/besynnerlig/shared_invite/enQtMzI3NzIyNTAzMjM1LTdmOGRhOTAwMmIwZWQ0MTNiZTU0MTY0MDk3OTVkYmYxYjE4NDE4MjcxMjg1YzAzNTJmZDM3NzJkYWU2ZDkwZmY) <--- Click link!

## License

This project is licensed under the [Eclipse Public License 1.0](https://opensource.org/licenses/EPL-1.0)

## Acknowledgments

* [Steve Bate](https://github.com/steve-bate) made [openHab2-jython](https://github.com/OH-Jython-Scripters/openhab2-jython), the great work from which **lucid** is derived from.

## Disclaimer
THIS SOFTWARE IS PROVIDED BY THE AUTHOR "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
