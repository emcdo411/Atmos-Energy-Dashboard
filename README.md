# AtmosEnergyDashboard

A Shiny dashboard application for managing customer data, incidents, and outages at Atmos Energy, built with R, Shiny, and SQL Server integration, and deployable to shinyapps.io.

---

## Table of Contents

- [Summary](#summary)
- [Features](#features)
- [Technologies Used](#technologies-used)
- [Code](#code)
- [Deployment to shinyapps.io](#deployment-to-shinyappsio)
- [Why This Matters](#why-this-matters)
- [Conclusion](#conclusion)

---

## Summary

The `AtmosEnergyDashboard` is a robust, interactive web application developed to streamline Atmos Energy's operational management. This project was a collaborative effort leveraging cutting-edge AI and database tools:

- **Grok AI (xAI)**: Played a pivotal role in iteratively designing, debugging, and refining the R Shiny application. Grok assisted in resolving SQL Server connection issues (e.g., Windows Authentication with `Trusted_Connection`), adapting the app for deployment (e.g., mocking data for shinyapps.io), and generating this README. Its real-time problem-solving and code generation capabilities accelerated development.
- **OpenAI**: While not directly used in this codebase, its influence is acknowledged as a benchmark for AI-driven development, inspiring the use of AI assistants like Grok to enhance productivity and innovation.
- **SQL Server Management Studio (SSMS)**: Served as the backbone for managing the `AtmosEnergyDB` database. SSMS was used to configure the database, ensure the `5CD147L68L\Veteran` Windows user had access, and verify table structures (e.g., `Customers`, `Incidents`, `Outages`) for seamless integration with the Shiny app.

The app started as a local RStudio project with live SQL Server connectivity and evolved into a deployable web app on shinyapps.io, overcoming challenges like authentication mismatches, package conflicts (e.g., `dateTimeInput` from `shinyWidgets`), and remote deployment constraints. The final product offers a mock-data version for public testing, with a clear path to reconnect to a hosted SQL Server database.

---

## Features

- **Home Dashboard**: Displays key metrics (total customers, outages, incidents) in visually appealing value boxes.
- **Customer Management**: Searchable table of customer data with real-time filtering.
- **Incident Logging**: Form to log new incidents with severity, location, and status, synced to a dynamic table.
- **Outage Tracking**: Form to record outages with date/time inputs, duration, and status, reflected in an interactive log.
- **Responsive Design**: Built with `shinydashboard` for a professional, user-friendly interface.
- **Deployment Ready**: Adapted for shinyapps.io with mock data, easily extensible to a live database.

---

## Technologies Used

- **R**: Core programming language for the app.
- **Shiny & shinydashboard**: Frameworks for building the interactive web interface.
- **DBI & odbc**: Libraries for SQL Server connectivity.
- **dplyr & DT**: Data manipulation and interactive tables.
- **ggplot2**: Included for potential future visualizations.
- **SQL Server**: Database management via SSMS.
- **shinyapps.io**: Hosting platform for deployment.
- **Grok AI**: Development assistant from xAI.

---

## Code

Below is the most recent version of the Shiny app (`AtmosEnergyShinyApp_NoDB.R`), designed with mock data for successful deployment to shinyapps.io:

```R
library(shiny)
library(shinydashboard)
library(dplyr)
library(DT)
library(ggplot2)

# Mock Data (Replace DB queries)
mockCustomerSummary <- data.frame(TotalCustomers = 100, CustomersWithOutages = 10, ActiveCustomers = 90)
mockIncidentSummary <- data.frame(TotalIncidents = 50, UnresolvedIncidents = 20, ResolvedIncidents = 30)
mockOutageSummary <- data.frame(TotalOutages = 25, OngoingOutages = 5, ResolvedOutages = 20)
mockCustomerData <- data.frame(CustomerID = 1:5, Name = c("Alice", "Bob", "Charlie", "Dana", "Eve"), AccountNumber = sprintf("ACC%03d", 1:5))
mockIncidentData <- data.frame(IncidentID = 1:3, CustomerID = c(1, 2, 3), IncidentDate = "2025-04-09", Severity = c("Low", "Medium", "High"), Location = "Area1", Description = "Test", Status = "New")
mockOutageData <- data.frame(OutageID = 1:2, Cause = c("Storm", "Maintenance"), StartTime = "2025-04-09 14:30", DurationHours = c(2, 4), AreaAffected = "Zone1", Status = "Ongoing")

# UI
ui <- dashboardPage(
  dashboardHeader(title = "Atmos Energy Dashboard (Mock)"),
  dashboardSidebar(
    sidebarMenu(
      menuItem("Home", tabName = "home", icon = icon("home")),
      menuItem("Customers", tabName = "customers", icon = icon("users")),
      menuItem("Incidents", tabName = "incidents", icon = icon("exclamation-triangle")),
      menuItem("Outages", tabName = "outages", icon = icon("bolt"))
    )
  ),
  dashboardBody(
    tabItems(
      # Home Screen
      tabItem(tabName = "home",
              fluidRow(
                valueBoxOutput("totalCustomers", width = 4),
                valueBoxOutput("outageCustomers", width = 4),
                valueBoxOutput("activeCustomers", width = 4)
              ),
              fluidRow(
                valueBoxOutput("totalIncidents", width = 4),
                valueBoxOutput("unresolvedIncidents", width = 4),
                valueBoxOutput("resolvedIncidents", width = 4)
              ),
              fluidRow(
                valueBoxOutput("totalOutages", width = 4),
                valueBoxOutput("ongoingOutages", width = 4),
                valueBoxOutput("resolvedOutages", width = 4)
              )
      ),
      # Customer Data Screen
      tabItem(tabName = "customers",
              fluidRow(
                box(
                  title = "Customer Data", width = 12, solidHeader = TRUE,
                  textInput("searchCustomer", "Search by Name or Account Number"),
                  DTOutput("customerTable")
                )
              )
      ),
      # Incident Report Screen
      tabItem(tabName = "incidents",
              fluidRow(
                box(
                  title = "Log New Incident", width = 4, solidHeader = TRUE,
                  selectInput("incidentCustomer", "Customer", choices = NULL),
                  dateInput("incidentDate", "Date", value = Sys.Date()),
                  selectInput("incidentSeverity", "Severity", choices = c("Low", "Medium", "High")),
                  textInput("incidentLocation", "Location"),
                  textAreaInput("incidentDesc", "Description"),
                  selectInput("incidentStatus", "Status", choices = c("New", "In Progress", "Resolved")),
                  actionButton("submitIncident", "Submit")
                ),
                box(
                  title = "Incident Log", width = 8, solidHeader = TRUE,
                  DTOutput("incidentTable")
                )
              )
      ),
      # Outage Report Screen
      tabItem(tabName = "outages",
              fluidRow(
                box(
                  title = "Log New Outage", width = 4, solidHeader = TRUE,
                  textInput("outageCause", "Cause"),
                  dateInput("outageDate", "Date", value = Sys.Date()),
                  textInput("outageTime", "Time (HH:MM)", value = format(Sys.time(), "%H:%M")),
                  numericInput("outageDuration", "Duration (Hours)", value = 0, min = 0),
                  textInput("outageArea", "Area Affected"),
                  selectInput("outageStatus", "Status", choices = c("Ongoing", "Resolved")),
                  actionButton("submitOutage", "Submit")
                ),
                box(
                  title = "Outage Log", width = 8, solidHeader = TRUE,
                  DTOutput("outageTable")
                )
              )
      )
    )
  )
)

# Server
server <- function(input, output, session) {
  # Reactive Values for Mock Data
  incidents <- reactiveVal(mockIncidentData)
  outages <- reactiveVal(mockOutageData)
  
  # Home Screen Summaries
  customerSummary <- reactive({ mockCustomerSummary })
  incidentSummary <- reactive({ mockIncidentSummary })
  outageSummary <- reactive({ mockOutageSummary })
  
  output$totalCustomers <- renderValueBox({
    valueBox(customerSummary()$TotalCustomers, "Total Customers", icon = icon("users"), color = "blue")
  })
  output$outageCustomers <- renderValueBox({
    valueBox(customerSummary()$CustomersWithOutages, "Customers with Outages", icon = icon("bolt"), color = "red")
  })
  output$activeCustomers <- renderValueBox({
    valueBox(customerSummary()$ActiveCustomers, "Active Customers", icon = icon("check"), color = "green")
  })
  output$totalIncidents <- renderValueBox({
    valueBox(incidentSummary()$TotalIncidents, "Total Incidents", icon = icon("exclamation"), color = "blue")
  })
  output$unresolvedIncidents <- renderValueBox({
    valueBox(incidentSummary()$UnresolvedIncidents, "Unresolved Incidents", icon = icon("hourglass"), color = "yellow")
  })
  output$resolvedIncidents <- renderValueBox({
    valueBox(incidentSummary()$ResolvedIncidents, "Resolved Incidents", icon = icon("check"), color = "green")
  })
  output$totalOutages <- renderValueBox({
    valueBox(outageSummary()$TotalOutages, "Total Outages", icon = icon("bolt"), color = "blue")
  })
  output$ongoingOutages <- renderValueBox({
    valueBox(outageSummary()$OngoingOutages, "Ongoing Outages", icon = icon("spinner"), color = "orange")
  })
  output$resolvedOutages <- renderValueBox({
    valueBox(outageSummary()$ResolvedOutages, "Resolved Outages", icon = icon("check"), color = "green")
  })
  
  # Customer Data Screen
  customerData <- reactive({
    data <- mockCustomerData
    if (nchar(input$searchCustomer) > 0) {
      data <- data %>% filter(grepl(input$searchCustomer, Name) | grepl(input$searchCustomer, AccountNumber))
    }
    data
  })
  output$customerTable <- renderDT({
    datatable(customerData(), options = list(pageLength = 10, scrollX = TRUE))
  })
  
  # Incident Report Screen
  observe({
    updateSelectInput(session, "incidentCustomer", choices = mockCustomerData$CustomerID)
  })
  observeEvent(input$submitIncident, {
    new_incident <- data.frame(
      IncidentID = max(incidents()$IncidentID) + 1,
      CustomerID = as.integer(input$incidentCustomer),
      IncidentDate = input$incidentDate,
      Severity = input$incidentSeverity,
      Location = input$incidentLocation,
      Description = input$incidentDesc,
      Status = input$incidentStatus
    )
    incidents(rbind(incidents(), new_incident))
    showNotification("Incident logged successfully!", type = "message")
  })
  output$incidentTable <- renderDT({
    datatable(incidents(), options = list(pageLength = 10, scrollX = TRUE))
  })
  
  # Outage Report Screen
  observeEvent(input$submitOutage, {
    startTime <- paste(input$outageDate, input$outageTime)
    new_outage <- data.frame(
      OutageID = max(outages()$OutageID) + 1,
      Cause = input$outageCause,
      StartTime = startTime,
      DurationHours = input$outageDuration,
      AreaAffected = input$outageArea,
      Status = input$outageStatus
    )
    outages(rbind(outages(), new_outage))
    showNotification("Outage logged successfully!", type = "message")
  })
  output$outageTable <- renderDT({
    datatable(outages(), options = list(pageLength = 10, scrollX = TRUE))
  })
}

# Run the app
shinyApp(ui, server)
```

---

## Deployment to shinyapps.io

Follow these steps to deploy the app to shinyapps.io:

1. **Install `rsconnect`**:
   ```R
   install.packages("rsconnect")
   ```

2. **Set Up shinyapps.io Account**:
   - Log into [shinyapps.io](https://www.shinyapps.io/).
   - Go to "Account" > "Tokens" and copy your `token` and `secret`.
   - In RStudio, configure your account:
     ```R
     library(rsconnect)
     rsconnect::setAccountInfo(name="your-username", token="YOUR_TOKEN", secret="YOUR_SECRET")
     ```

3. **Prepare the App**:
   - Save the code above as `app.R` in your project directory.

4. **Deploy the App**:
   - Run this in RStudio:
     ```R
     rsconnect::deployApp()
     ```
   - Monitor the deployment log for success (e.g., "Application successfully deployed to https://your-username.shinyapps.io/app-name/").

5. **Verify**:
   - Visit the provided URL to interact with the deployed app.

**Note**: This version uses mock data. For live database integration, host `AtmosEnergyDB` (e.g., on Azure SQL), update `dbConnect` with SQL Server Authentication, and set environment variables in shinyapps.io for credentials.

---

## Why This Matters

The `AtmosEnergyDashboard` isn’t just a technical exercise—it’s a transformative tool for energy management:

- **Operational Efficiency**: Centralizes customer, incident, and outage data, reducing manual tracking and response times.
- **Scalability**: Designed to transition from local to cloud-hosted databases, supporting Atmos Energy’s growth.
- **Innovation**: Demonstrates how AI (Grok) and modern tools (Shiny, SQL Server) can solve real-world problems, setting a precedent for tech-driven energy solutions.
- **Accessibility**: Deployment on shinyapps.io makes it accessible anywhere, empowering remote teams and stakeholders.

This project showcases the power of integrating AI-assisted development with traditional database systems, offering a blueprint for utilities to modernize operations while maintaining reliability.

---

## Conclusion

The `AtmosEnergyDashboard` is a testament to collaborative innovation, blending Grok AI’s problem-solving prowess, SSMS’s robust database management, and Shiny’s interactive web capabilities. From overcoming local connection hurdles to achieving a deployable web app, this project delivers a functional dashboard ready for real-world use—whether with mock data for demonstration or a live database for production. Clone this repo, explore the code, and adapt it to your needs—let’s keep pushing the boundaries of energy management together!

---
