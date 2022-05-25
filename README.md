# weather

This is a single-page web app built with Vue.

The app allows the user to look up current weather data for any location by entering either City / State / Country or latitude / longitude.  It accesses the openweathermap.org API and returns current weather info.

If the user executes a search based on longitudinal coordinates, the app makes one call to the weather endpoint.  However, if the user enters city information instead, the app makes two calls:  the first to convert city info into coordinates, the second to get weather info based on those coordinates.