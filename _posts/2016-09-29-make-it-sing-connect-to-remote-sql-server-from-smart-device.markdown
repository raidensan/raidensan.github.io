---
published: true
title: Make It Sing: Connect to remote SQL Server from Smart Device
layout: post
tags: [sql-ce, sql-server, smart-device, windows-ce, windows-mobile]
---
In this post I'll demonstrate how to connect to remote SQL Server from Smart Device projects.

What you'll need:

* Visual Studio 2005/2008 Professional (Ideally with all available SP's installed)
* Windows CE/Mobile SDK
* [SQL Server Compact Edition 3.5 SP2](https://www.microsoft.com/en-us/download/details.aspx?id=5783). Remember, this is for desktop.
* A running SQL Server instance

_Note: SQL Server instance must be configured to accept incoming connection. Also a make sure that firewall or antivirus does not block SQL Server connection port (Usually 1433)._

Run VisualStudio 2008 and create a new SmartDevice project.

In the next screen select Windows CE as target platform and choose .Net Compact Framework 3.5. 

Finally, select project type, e.g. Device Application.

Now add a reference to SqlClient.dll in 'C:\Program Files (x86)\Microsoft SQL Server Compact Edition\v3.5\Devices\Client\System.Data.SqlClient.dll'

_Note: As you can see SqlClient.dll is inside SQL Server Compact's binaries for Device. Naturally, this is not for Desktop._

The following is a sample for creating and opening connection:

	using(SqlConnection conn = new SqlConnection(GetConnectionString()))
	{
		try
		{
			if(conn.ConnectionState != ConnectionState.Open)
				conn.Open();
		}
		catch(SqlException e)
		{
			// Log error, etc.
		}
	}
