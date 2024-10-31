```mermaid

    C4Container
    title Service oriented app
    
    Person(customer, Customer, "A customer in front of the laptop Has root CA-cert")

    Container_Boundary(c2, "Application services. Has root CA-cert") {
        Container(app1_service, "Back End service", "", "")
        Container(app2_service, "Back End service", "", "")
    }

    Container_Boundary(c1, "Database Rest API Has root CA cert") {
       
        Container(appdbservice, "Rest API", "", "Ineract with  database only")
        ContainerDb(database, "Database", "SQL Database")
       

    }

    Rel( customer, app1_service, "user interacttion","TLS with Server Auth", "1")
    Rel( customer, app2_service, "user interacttion","TLS with Server Auth", "2")
    
    Rel(app1_service,  "appdbservice", "TLS with Client Auth", "3")
    Rel(app2_service,  "appdbservice", "TLS with Client Auth", "4")

    BiRel(appdbservice,  "database", "sql-net", "5")

    Rel(customer,  "appdbservice", TLS with Server Auth , "Not aloud", "0")

    UpdateRelStyle(customer,  "appdbservice" $textColor="red",$lineColor="red", $offsetY="-80", $offsetX="-120")


    UpdateRelStyle(customer, app1_service, $textColor="blue",$lineColor="blue", $offsetY="-40", $offsetX="40")
    UpdateRelStyle(customer, app2_service, $textColor="blue",$lineColor="blue", $offsetY="-140", $offsetX="-140")
    
    UpdateLayoutConfig($c4ShapeInRow="1", $c4BoundaryInRow="2")

```