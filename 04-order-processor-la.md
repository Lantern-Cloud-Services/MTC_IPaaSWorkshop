# Exercise 4 - Order PRocessor Logic App

In the exercise we'll create the Logic App that is responsible for processing the recieved order document that has been persisted in the "PendingOrders" collection.  The Logic App is implemented as an HTTP request trigger that will recieve the conceptually completed order document.  It will persist the order to the "ProcessedOrders" collection and delete the preprocessed order document.  We'll again front the API with APIM and test/ Later we'll modify this Logic App to also put the completed order onto a Service Bus queue to continue processing.

1) Create the order processor Logic App:

    ![Order Processor Logic App](./media/ex4/la2_op.png)

2) Create the HTTP request trigger:

    ![HTTP Trigger](./media/ex4/la2_op_httpreq.png)

    - Use the following sample payload representing a completed order to create the JSON schema:

            {
                "id":"b878b264-0b22-484e-a564-484ad1dc6f91",
                "product":"Grape Paper",
                "quantity":100,
                "email":"jasoncoo@onemtc.net",
                "address":"123 Somewhere Ln.",
                "firstName":"John",
                "lastName":"Doe"
            }

        
    - The request method is POST

3) Parse the JSON of the incoming HTTP request body to paramaterize each element of the JSON document:

    ![Parse JSON Action](./media/ex4/la2_op_pjson.png)

    - Use the same sample payload from the http request but replace the values in the JSON with dynamic properties from the HTTP request.

4) Create an action to create or update a document, this will persist the newly created completed order document:

    **Note:** when using a prev V3 action the "Partition Key Value" is required.

    Pre V3

    ![Pre V3 Create or Update Action](./media/ex4/la2_op_cdb.png)

    V3

    ![V3 Create or Update Action](./media/ex4/la2_op_cdbv3.png)

    **Note:** Consider using RBAC for Cosmos authorization.

            $resourceGroupName = "<myResourceGroup>"
            $accountName = "<myCosmosAccount>"
            
            Get-AzCosmosDBSqlRoleDefinition -AccountName $accountName -ResourceGroupName $resourceGroupName
            $roleDefinitionId ="<cosmosRewadWriteRole>"
            $principalId = "<logicAppPrinicpalId>"            

            New-AzCosmosDBSqlRoleAssignment -AccountName $accountName `
                -ResourceGroupName $resourceGroupName `
                -RoleDefinitionId $roleDefinitionId `
                -Scope "/" `
                -PrincipalId $principalId           

    - The Database/Collection IDs are FlavoredOfficeSupplies/ProcessedOrders
    - The document to be persisted should match the structure of the inbound rest JSON document and with values dynamically generated from the JSON action as pictured.
    

5) Next delete the pre processed order document from the "PendingOrders" collection, use the Document ID from the previous consmos action:

    ![Delete Object Action](./media/ex4/la2_op_cdbdel.png)


6) Create an HTTP Response action and provide a relevant response message:

    ![Response Action](./media/ex4/la2_op_res.png)

7) Save and test the Logic App again using Postman:

    a) Take the values from an existing pre processed order document in the "PreProcessed" Cosmos collection and fill in the additional values:  
    
    ![Existing Order](./media/ex4/la2_op_test1.png)

    b) Submit the request to the Logic App endpoint using Postman:

    ![Postman Request](./media/ex4/la2_op_test2.png)

    - Confirm that you receive a valid response.

    c) Using the Cosmos Data Explorer, confirm that the pending order with the same ID has been deleted from the "PendingOrders" collection and that the new completed order has been persisted to the "ProcessedOrders" collection:

    ![Process Orders Collection](./media/ex4/la2_op_test3.png)

8) Integrate the Logic App with APIM.

    a) Begin by cloning the existing Orders API Create Order operateion.  You could also choose to implement the new process order API as a it own first order API.  This highlights a philosophical choice when structuring you APIs, you can either group only REST verbs under the same API or all related operations.  In this case we're grouping all operations.

    ![Process Orders Collection](./media/ex4/la2_op_apim1.png)

    b) Rename the operation Process Order and adjust the frontend URL path as shown.

    ![Process Orders Collection](./media/ex4/la2_op_apim2.png)

    c) Repoint the API to the new Order Processor Logic App

    ![Process Orders Collection](./media/ex4/la2_op_apim3.png)

    d) The final state of the Orders API

    ![Process Orders Collection](./media/ex4/la2_op_apim4.png)

9) Test the new operation in the APIM admin portal (after using the Create Order API to generate a new order).

    a) Retrieve the ID of the existing order

    ![Process Orders Collection](./media/ex4/la2_op_test_p1.png)

    b) Add the new processed order values:

        {
            "id":"85b62a01-bea3-4b8d-986a-152bafd363f4",
            "product":"Grape Paper",
            "quantity":101,
            "email":"jasoncoo@onemtc.net",
            "address":"123 Somewhere Ln.",
            "firstName":"John",
            "lastName":"Doe"
        }

    c) Submit the payload to the Process Order API:

    ![Process Orders Collection](./media/ex4/la2_op_test_p2.png)    

    ![Process Orders Collection](./media/ex4/la2_op_test_p3.png)

    d) Confirm the new processed order document has been added to the ProcessedOrders collection and removed from the Pending Orders collection.

    ![Process Orders Collection](./media/ex4/la2_op_test_p4.png)

The order Processing Logic App is now complete and will persist the new processed order document to the ProcessedOrders collection as well as remove the original document from the PendingOrders collection.  Continue to [Exercise 5: Create Integration Trigger Function App](./05-integration-trigger-function.md) to implement the function app that will trigger on new order creation.
