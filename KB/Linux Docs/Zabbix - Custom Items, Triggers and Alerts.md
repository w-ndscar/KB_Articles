
For an instance we'll create an item to get the free space of a disk drive (Windows). Then we'll create a trigger and an alert if it goes below a certain value.
### Creating an item

Ref: https://www.zabbix.com/documentation/current/en/manual/config/items/item

- Go to Data Collection → Hosts
- Click on **Items** in the row of the host at the top
- Click on **Create item** in the upper right corner of the screen

	***Example:***
- Name: (D:) Free Space
- Key: `vfs.fs.size[D:,free]`
- Check the **Enabled** check box
- Click on **Test**. Then click on **Get value and Test**. Check if the value is correct.
- Then click on **Save**
- To convert the Bytes value to GB, go to the **Preprocessing** Tab, select **Custom Multiplier** under Preprocessing Steps and enter the value: ***9.3132257461548E-10***

### Creating a trigger

Ref: https://www.zabbix.com/documentation/current/en/manual/config/triggers/trigger

- Go to: **Data collection** → ***Hosts***
- Click on ***Triggers*** in the row of the host
- Click on ***Create trigger*** to the right (or on the trigger name to edit an existing trigger)

	***Example:***
- Name: (D:) Disk space is low
- Expression: Click on Add. Select the item: (D:) Free Space. Select last() for function and in Result select <= and enter the desired value in GB (Already converted the value from Bytes to GB in the previous step).
- This will trigger the event if free space of the D drive is less than 5 GB.
- Select the severity as High

### Creating an Action

Ref: https://www.zabbix.com/documentation/current/en/manual/config/notifications/action/conditions
*Note:* For Email alerts, it is necessary to configure the SMTP settings (**FROM** email address) under Alerts → Media Types and the (**TO**) email address on the user profile in Zabbix by going to Users → Users and select the user and go to Media tab at the top. Click on Add and Type Office365 or Gmail or whatever the (**TO**) email is and specify the email.

- Go to: **Alerts** → ***Actions*** → ***Trigger actions***
- Click on **Create action** at the top right

	***Example:***
- Name: D: Free Space
- Conditions: Click **Add** and select the Trigger that was created before ((D:) Disk space is low)
- Click on **Operations** Tab. Under **Operations** click on **Add**
- Under ***Send to users***, select the appropriate user. In this case: Admin
- Select Office365, under **Send only to**.
- Check the **Custom message** checkbox and provide a custom subject and message
- Then click on **Add**.

### Grant the Host Permission for the Group

- Go to **Users** → **User Groups** and select **Host permissions** at the top row
- Click on **Select** and select the User Group and select Read
- Then click on **Add** and click **Update**
