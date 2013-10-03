Enter file contents here/**
 * Away with mode types
 *
 *  Author: brian@bevey.org
 *  Date: 9/10/13
 *  Updated by Sean Stozki - 10-2-2013
 *
 *  Monitors a set of presence detectors and triggers a mode change when everyone has left.
 *  When at least one person returns home, set the mode back to Home (or whatever is defined).
 *  The mode is determined based on the Sunrise and Sunset
 */

preferences {
  section("When all of these people leave home") {
    input "people", "capability.presenceSensor", multiple: true
  }

  section("Change to this mode to...") {
    input "newAwayMode", "mode", title: "Everyone is away"
    input "newHomeMode", "mode", title: "At least one person home"
	input "newNightMode", "mode", title: "At least one person home at night"
  }
  section("What is your zip code...") {
	input ('zip', 'number', title: 'Zip Code')
	}

  section("False alarm threshold (defaults to 10 min)") {
    input "falseAlarmThreshold", "decimal", title: "Number of minutes", required: false
  }

  section( "Notifications" ) {
    input "sendPushMessage", "enum", title: "Send a push notification?", metadata:[values:["Yes","No"]], required:false
  }
}

def installed() {
  log.debug "Installed with settings: ${settings}"
  log.debug "Current mode = ${location.mode}, people = ${people.collect{it.label + ': ' + it.currentPresence}}"
  subscribe(people, "presence", presence)
}

def updated() {
  log.debug "Updated with settings: ${settings}"
  log.debug "Current mode = ${location.mode}, people = ${people.collect{it.label + ': ' + it.currentPresence}}"
  unsubscribe()
  subscribe(people, "presence", presence)
}

def presence(evt) {
  log.debug("evt.name: ${evt.value}")
  if (evt.value == "not present") {
    if (location.mode != newAwayMode) {
      log.debug("Checking if everyone is away")

      if (everyoneIsAway()) {
        log.info("Starting ${newAwayMode} sequence")
        def delay = (falseAlarmThreshold != null && falseAlarmThreshold != "") ? falseAlarmThreshold * 60 : 10 * 60 
        runIn(delay, "setAway")
      }
    }

    else {
      log.debug("Mode is the same, not evaluating")
    }
  }

  else {
    if (location.mode != newHomeMode) {
      log.debug("Checking if anyone is home")

      if (anyoneIsHome()) {
        log.info("Checking for Mode...")
        setMode()
      }
    }

    else {
      log.debug("Mode is the same, not evaluating")
    }
  }
}

def setAway() {
  if (everyoneIsAway()) {
    def message = "${app.label} changed your mode to '${newAwayMode}' because everyone left home"
    log.info message
    send(message)
    setLocationMode(newAwayMode)
  }

  else {
    log.info("Somebody returned home before we set to '${newAwayMode}'")
  }
}

def setMode(evt) {
    if (anyoneIsHome()){
	def data = getWeatherFeature('astronomy', settings.zip as String)
    
    def sunsetTime = data.moon_phase.sunset.hour + ':' + data.moon_phase.sunset.minute
    def sunriseTime = data.moon_phase.sunrise.hour + ':' + data.moon_phase.sunrise.minute
    def currentTime = data.moon_phase.current_time.hour + ':' + data.moon_phase.current_time.minute
    
    def localData = getWeatherFeature('geolookup', settings.zip as String)
    
    def timezone = TimeZone.getTimeZone(localData.location.tz_long)
    
	log.debug( "Sunset today is at $sunsetTime" )
	log.debug( "Sunrise today is at $sunriseTime" )


	//log.debug ("$evt.value: $evt, $settings")
 	def startTime = timeToday(sunsetTime, timezone)
    def endTime = timeToday(sunriseTime, timezone) 
	if (now() < startTime.time && now() > endTime.time) 
    {
     def message = "${app.label} changed your mode to '${newHomeMode}' because somebody came home"
	 log.info message
     send(message)
	 setLocationMode(newHomeMode)
    }
    else {
         log.trace ("Going into Night Mode: $newNightMode")
	     setLocationMode(newNightMode)
    }
}
}
private everyoneIsAway() {
  def result = true
  for (person in people) {
    if (person.currentPresence == "present") {
      result = false
      break
    }
  }

  log.debug("everyoneIsAway: ${result}")
  return result
}

private anyoneIsHome() {
  def result = false
  for (person in people) {
    if (person.currentPresence == "present") {
      result = true
      break
    }
  }

  log.debug("anyoneIsHome: ${result}")
  return result
}

private send(msg) {
  if (sendPushMessage != "No") {
    log.debug("Sending push message")
    sendPush(msg)
  }

  log.debug(msg)
}
