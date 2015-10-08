# Using smartphone GPS and IP address location in a Shiny app

In a shiny app it might be advantageous to have access to the location of the user, for example to set the initial view of a map to their location, or to serve up information from their local area.

[Click here for a demonstration using the SuperZip example](https://tomaugust.shinyapps.io/shiny_geolocation)

## Javascript

First we use the `tags$script()` function in shiny to add a bit of JavaScript to our shiny app. This script goes in the `ui.R` script, and should be placed inside a panel that is displayed when the user first visits your app. This ensures the script is run when the app starts. Using this script we assign three things to input variables. 

1. `input$lat` - Latitude (numeric)
2. `input$long` - Longitude (numeric)
3. `input$geolocation` - Geolocation (logical)

```r
tags$script('
  $(document).ready(function () {
    navigator.geolocation.getCurrentPosition(onSuccess, onError);

    function onError (err) {
    Shiny.onInputChange("geolocation", false);
    }
    
   function onSuccess (position) {
      setTimeout(function () {
          var coords = position.coords;
          console.log(coords.latitude + ", " + coords.longitude);
          Shiny.onInputChange("geolocation", true);
          Shiny.onInputChange("lat", coords.latitude);
          Shiny.onInputChange("long", coords.longitude);
      }, 1100)
  }
  });
')
```

When we run the app the user will be asked to share their location. Until they do all three input parameters will be `NULL`. If the user allows their browser access to their location latitude and longitude will become numeric values and geolocation will become `TRUE`. If they do not allow geolocation, latitude and longitude will become `NULL` and geolocation will become `FALSE`.

Here is a minimal example for you to try out. It is also in the folder simple-example in this repository.

### ui.r

```r
library(shiny)

shinyUI(fluidPage(

  titlePanel("Using Geolocation"),
    
  tags$script('
      $(document).ready(function () {
        navigator.geolocation.getCurrentPosition(onSuccess, onError);
              
        function onError (err) {
          Shiny.onInputChange("geolocation", false);
        }
              
        function onSuccess (position) {
          setTimeout(function () {
            var coords = position.coords;
            console.log(coords.latitude + ", " + coords.longitude);
            Shiny.onInputChange("geolocation", true);
            Shiny.onInputChange("lat", coords.latitude);
            Shiny.onInputChange("long", coords.longitude);
          }, 1100)
        }
      });
              '),
    
    # Show a plot of the generated distribution
    fluidRow(column(width = 2,
                    verbatimTextOutput("lat"),
                    verbatimTextOutput("long"),
                    verbatimTextOutput("geolocation"))
    )
  )
)
```

### server.r

```r
library(shiny)

shinyServer(function(input, output) {
  
  output$lat <- renderPrint({
    input$lat
  })
  
  output$long <- renderPrint({
    input$long
  })

  output$geolocation <- renderPrint({
    input$geolocation
  })
  
})
```

## SuperZip example

In the [SuperZip example](https://tomaugust.shinyapps.io/shiny_geolocation) (borrowed from the [Shiny gallery](http://shiny.rstudio.com/gallery/superzip-example.html)) I have implemented the geolocation feature so that when the page loads the view will be updated to show the user's location. This is achieved by adding the JavaScript to the `ui.r` script as described above and using an `observe` statement to update the view when we have to users location.

```r
  # Zoom in on user location if given
  observe({
    if(!is.null(input$lat)){
     map <- leafletProxy("map")
     dist <- 0.5
     lat <- input$lat
     lng <- input$long
     map %>% fitBounds(lng - dist, lat - dist, lng + dist, lat + dist)
    }
  })
```

### Acknowledgements

Many thanks to @kazlauskis for writing the Javascript and help with debugging
