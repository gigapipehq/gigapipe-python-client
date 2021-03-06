# Gigapipe Python Client SDK

## What is Gigapipe?
Based on a managed ClickHouse service, Gigapipe offers a customisable ecosystem to leverage your Big Data. Select as many or as few services as you require to either integrate with your existing infrastructure, or launch entirely through Gigapipe.
Gigapipe is simple. Select your region and configure your machine type in GCP or AWS, select the managed services you want and launch! Gigapipe will deploy your fully managed ClickHouse Cluster and additional services which you can then plug directly into your existing infrastructure.
Gigapipe is fully managed, easily scalable and incredibly cost-effective.

_Should you require further information, do not hesitate to visit the [Gigapipe API Docs](https://docs.gigapipe.com/api/introduction)._

## What is this library for?
This is the official Gigapipe API Library for Python. It supports Python 3.x.

## Quick start
Install the Pypi package
```
pip install -U gigapipe
```
### The Gigapipe instance
Import the gigapipe library into your project, attach your secret key to it, and create an instance using your client id:
```python
import gigapipe

# Add your client secret key to the library
gigapipe.client_secret = "your_client_secret"

# Create an instance using your client id
gigapipe_client = gigapipe.GigapipeClient(client_id="your_client_id")
```
That's it, this is all you need to start using the Gigapipe API!

### Basic Elements

#### Use your Gigapipe instance to make calls to the library.

- Obtain the providers
> Returns the list of Gigapipe providers
```python
providers = gigapipe_client.root.get_providers()

# Payload response
[
    {
        "id": "1",
        "name": "AWS",
    }
    ...
]
```

- Obtain the regions
> Returns all the available regions for a given provider
```python
regions = gigapipe_client.root.get_regions(provider_id=1)

# Payload response
[
    {
        "id": 1,
        "name": "Ashburn, Virginia", 
        "code": "us-east-1",  
        "provider": {
            "id": 1, 
            "name": "AWS"
        }
    }
    ...
]

```
- Obtain the machines
> Returns all the available machines per provider and region
```python
machines = gigapipe_client.root.get_machines(provider_id=1, region_id=1)

# Payload response
[
    {
        "id": 1, 
        "ram": 8, 
        "cpu": 2, 
        "price": 0.00336, 
        "name": "m5_large", 
        "provider": {
            "id": 1, 
            "name": "AWS"
        }, 
        "region": {
            "name": "Ashburn, Virginia", 
            "code": "us-east-1"
        }, 
        "ram_unit": "Binary"
    }
    ...
]
```

- Obtain a machine
> Given an id, it returns a machine
```python
machine = gigapipe_client.root.get_machine(machine_id=1)

# Payload response
{
    "id": 1, 
    "ram": 8, 
    "cpu": 2, 
    "price": 0.00336, 
    "name": "m5_large", 
    "provider": {
        "id": 1, 
        "name": "AWS"
    }, 
    "region": {
        "name": "Ashburn, Virginia", 
        "code": "us-east-1"
    }, 
    "ram_unit": "Binary"
}
```

- Obtain the disk types
> Returns all the disk types per provider and region
```python
response = gigapipe_client.root.get_disk_types(provider_id=1, region_id=1)

# Payload response
[
    {
        "name": "gp2", 
        "price": 0.0000003321, 
        "unit": "Decimal"
    }
    ...
]
```

### The Users API

#### Use your Gigapipe instance to make calls to the library.
- Obtain your user info
> Returns non-sensitive user info
```python
user_info = gigapipe_client.users.get_info()

# Payload response
{
    "first_name": "John",
    "last_name": "Doe",
    "organization": {
        "name": "Heindl"
    },
    "verified": true,
    "email": "john@doe.com"
}
```

- Change your password
> Allows the user to change their password
```python
gigapipe_client.users.change_password({
    "new_password": "Your new password",
    "new_password_verification": "Your new password",
    "old_password": "Your old password"
})
```

- Update first and last name
> Allows the user to change their first and last name
```python
gigapipe_client.users.update_name({
    "first_name": "John",
    "last_name": "Doe",
})
```

- Obtain the upcoming invoice
> Note that the invoice belongs to the organization despite getting it from the user 
```python
user_info = gigapipe_client.users.get_upcoming_invoice()

# Payload response
{
    "amount_due": 0.0,          # What your organization owes this month so far
    "amount_spent": 0.0,        # What your organization has spent this month so far
    "credits_left": -60000.0,   # Remaining credits
    "credits_live": -60000.0    # Used credits
}
```

- Obtain your permissions
> Returns the list of permissions of the user in session
```python
user_info = gigapipe_client.users.get_permissions()

# Payload response
[
    {
      "type": "EDIT_CLUSTERS",
      "name": "edit_clusters",
    },
    {
      "type": "MANAGE_BILLING",
      "name": "manage_billing",
    }
    ...
]
```

- Delete your user
> Note that, if you delete your user using the Gigapipe Client SDK, you will no longer be able to use it and, your program will not be capable of making more requests to the API as authentication will be lost.
```python
gigapipe_client.users.delete_user()

# Payload response when the user is successfully deleted
{
  "payment_link": null,
  "account_deleted": true,
    ...
}

# Payload response when the user cannot be deleted because the company has 
# unpaid invoices (only for owners)
{
  "payment_link": "https://invoice_link...",
  "account_deleted": false,
    ...
}
```

### The Organizations API

#### Use your Gigapipe instance to make calls to the library.

- Obtain the users
> Returns all the users of the organization the user in session belongs to
```python
users = gigapipe_client.organizations.get_users()

# Payload response
[
    {
        "first_name": "John", 
        "last_name": "Doe", 
        "email": "john@doe.com", 
        "role": "Owner"
    }
    ...
]
```

- Obtain the invites
> Returns the list of invites sent by users of your organization
```python
invites = gigapipe_client.organizations.get_invites()

# Payload response
[
    {
        "email": "user@invited.conm"
    }
    ...
]
```

- Obtain the upcoming invoice
> Returns the upcoming invoice your organization will pay at the end of the current month
```python
gigapipe_client.organizations.get_upcoming_invoice()

# Payload response
{
    "amount_due": 0.0,          # What the organization owes this month so far
    "amount_spent": 0.0,        # What the organization has spent this month so far
    "credits_left": -60000.0,   # Remaining credits
    "credits_live": -60000.0    # Used credits
}
```

- Delete organization
> Note that, when the organization is deleted all its users will be gone as well, the unpaid invoices will be paid automatically using the customer credit card and, the subscription will be cancelled.
> If you delete your organization using the Gigapipe Client SDK, you will no longer be able to use it and, your program will not be capable of making more requests to the API as authentication will be lost.
```python
gigapipe_client.organizations.delete_organization()

# Payload response when the organization is deleted successfully
{
    "payment_link": null,
    "organization_deleted": true
}

# Payload response when there are unpaid invoices and the system 
# could not charge the invoice to the current credit card
{
    "payment_link": "some_payment_link",
    "organization_deleted": false
}
```

### The Invites API

#### Use your Gigapipe instance to make calls to the library.

- Send an invitation
> Sends an email that contains an invitation for a user to sign up on Gigapipe and join the organization
```python
gigapipe_client.invites.send_invite({
    "email": "laura@tesla.com",
    "organization_name": "Tesla",
})
```

- Obtain an invitation
> Obtains an invitation that has been sent to a user
```python
invite = gigapipe_client.invites.get_invite(token="your_invite_token")

# Payload response
{
    "email": "laura@tesla.com",
    "organization_name": "Tesla",
    "organization_slug": "tesla"
}
```

- Delete an invitation
> Revokes an invitation so that the user receiving it is no longer allowed to use it
```python
gigapipe_client.invites.delete_invite("laura@tesla.com")
```

### The Roles API

#### Use your Gigapipe instance to make calls to the library.

- Change the user role
> If enough permissions, it changes the role of another user inside the organization. e.g. Turn a Member into an Admin
```python
gigapipe_client.roles.switch({
    "user_email": "laura@tesla.com",
    "role_name": "Admin"  # (Owner, Admin, Member) 
})
```

### The Clusters API

#### Use your Gigapipe instance to make calls to the library.

- Create a cluster
> Note that the cluster isn't immediately available upon creation. It will take some time for Kubernetes to have it ready and running, hence the message: 'Cluster creation in progress.' 
> Feel free to query the cluster after a few minutes, when it will for sure be ready.
```python
gigapipe_client.clusters.create_cluster({
    "name": "Cluster Test",
    "machine_id": 1,
    "clickhouse": {
        "version_id": 1,
        "shards": 3,
        "replicas": 1,
        "disks": [
            {
                "name": "string",  # Optional (The first disk does not accept a custom name)
                "size": 150.0,
                "unit": "GB",
                "type": {
                    "name": "gp2"
                },
                "autoscaling": False
            }
        ],
        "admin": {
            "username": "john",
            "password": "john-pw"
        }
    },
    "provider_id": 2,
    "region_id": 4
})

# Payload response
{
    "message": "Cluster creation in progress."
}
```

- Cluster Query
> After having been waiting a few minutes for its creation, it's time to query the cluster and see that it got created and it's ready and running
>
> To do so, not only should the cluster slug be passed as a parameter but the clickhouse query as well, which has to be a string in the Clickhouse format. In this example the query 'SELECT now()' will be used.
```python
gigapipe_client.clusters.query_cluster(
    cluster_slug="cluster-john", 
    query="SELECT now()"
)

# Payload response
{
    "meta": [
        {
            "name": "now()", 
            "type": "DateTime"
        }
    ], 
    "data": [
        {
            "now()": "2022-02-14 13:20:38"
        }
    ], 
    "rows": 1, 
    "statistics": {
        "elapsed": 0.000865866, 
        "rows_read": 1, 
        "bytes_read": 1
    }
}
```

- Cluster Metadata
> Returns a dictionary containing the cluster metadata
```python
gigapipe_client.clusters.get_metadata(cluster_slug="cluster-john")

# Payload response
{
    "endpoint": "clickhouse_url", 
    "grafana_endpoint": "grafana_url", 
    "rows": 0, 
    "disks": [
        {
            "name": "default", 
            "type": "local", 
            "free_space": 2693251072, 
            "total_space": 18211586048
        }, 
        ...
    ]
}
```

- Get all the clusters
> Returns a list of all the clusters created by members of the organization
```python
gigapipe_client.clusters.get_clusters()

# Payload response
[
    {
        "name": "Cluster Test", 
        "slug": "cluster-john", 
        "region": {
            "name": "Ashburn, Virginia", 
            "code": "us-east-1"
        }, 
        "provider": {
            "id": 2, 
            "name": "AWS"
        }, 
        "status": "Active", 
        "shards": 3, 
        "replicas": 1, 
        "user": {
            "first_name": "John", 
            "last_name": "Doe"
        }, 
        "machine": {
            "ram": 8, 
            "cpu": 2, 
            "id": 1, 
            "ram_unit": "Binary"
        }, 
        "disks": [
            {
                "autoscaling": false, 
                "type": {
                    "name": "gp2", 
                    "price": 0.00000000045, 
                    "unit": "Decimal"
                }, 
                "size": 150.0, 
                "unit": "GB"
            }
        ], 
        "created_at": "2022-02-14T10:11:26.567713"
    }
    ...
]
```

- Get cluster
> Given a cluster slug, it returns all its info
```python
cluster = gigapipe_client.clusters.get_cluster(cluster_slug="cluster-john")

# Payload response

{
    "name": "Cluster Test", 
    "slug": "cluster-test", 
    "region": {
        "name": "Ashburn, Virginia", 
        "code": "us-east-1"
    }, 
    "provider": {
        "id": 2, 
        "name": "AWS"
    }, 
    "status": "Active", 
    "shards": 3, 
    "replicas": 1, 
    "user": {
        "first_name": "John", 
        "last_name": "Doe"
    }, 
    "machine": {
        "ram": 8, 
        "cpu": 2, 
        "id": 1, 
        "ram_unit": "Binary"
    }, 
    "disks": [
        {
            "autoscaling": false, 
            "type": {
                "name": "gp2", 
                "price": 0.00000000045, 
                "unit": "Decimal"
            }, 
            "size": 150.0, 
            "unit": "GB"
        }
    ], 
    "created_at": "2022-02-14T10:11:26.567713"
}
```

- Stop Cluster
> Stopping the cluster doesn't involve getting rid of it. The disks and the data in them will safely be kept, whereas the machine is stopped.
> In other words, the organization will still be charged in terms of disks but not machines.
```python
gigapipe_client.clusters.stop_cluster(cluster_slug="cluster-test")

# Payload response
{
    "message": "Stopping cluster <cluster-test>..."
}
```

- Resume Cluster
> Resuming the cluster involves getting it back to normal by restarting its machine. As of that moment, the organization is fully charged yet again (disks and machines).
```python
gigapipe_client.clusters.resume_cluster(cluster_slug="cluster-test")

# Payload response
{
    "message": "Resuming cluster <cluster-test>..."
}
```

- Scale Cluster
> Adds shards and replicas to an existing cluster
```python
gigapipe_client.clusters.scale_nodes("cluster-test", payload={
    "new_shards": 1,
    "new_replicas": 1
})

# Payload response
{
    "message": "Cluster scaling in progress."
}
```

- Add disks
> Adds disks to a cluster
```python
gigapipe_client.clusters.add_disks("cluster-test", payload=[{
    "name": "your_disk_name",
    "autoscaling": True,
    "type": "gp2",
    "size": 10.0,
    "unit": "GB"
}])

# Payload response
{
    "message": "Adding disks to cluster <cluster-test>..."
}
```

- Change Machine
> Changes the machine of a cluster
```python
gigapipe_client.clusters.change_machine("cluster-test", machine_id=2)

# Payload response
{
    "message": "Changing machine in cluster <cluster-test>..."
}
```

- Disk Expansion
> Expands a disk base on a cluster a disk id and a size
```python
gigapipe_client.clusters.expand_disk('cluster2', disk_id=1, payload={
    'size': 10.0
})

# Payload response
{
    "message": "Expanding disk 1 on cluster <cluster-test>..."
}
```

- Cluster costs
> The list of all the costs per cluster 
```python
gigapipe_client.clusters.get_costs()

# Payload response
[
    {
        "slug": "cluster-test", 
        "cost": 0.026882283105023334
    }
    ...
]
```

- Transfer Cluster
> Transfers the cluster to another user of the organization as long as they have permission to hold it
```python
gigapipe_client.clusters.transfer_cluster(
    cluster_slug="cluster-test", 
    email="target_user@gmail.com"
)

# Payload response
{
    "message": "Cluster <cluster-test> transferred to <target_user@gmail.com>"
}
```

- Delete Cluster
> Given a slug, this method deletes a cluster
```python
gigapipe_client.clusters.delete_cluster(cluster_slug="cluster-test")

# Payload response
{
    "message": "Cluster deletion in progress.",
    ...
}
```

- Change ClickHouse version of Cluster
> Given a slug and a ClickHouse version id, this method changes the ClickHouse version of a cluster
```python
gigapipe_client.clusters.change_clickhouse_version("cluster-test", payload={"id": 1})

# Payload response
{
    "message": "Changing ClickHouse version in cluster 'cluster-test'...",
    ...
}
```

- Change table engine to ReplicatedMergeTree type
> Given a cluster slug and a table name, change a table engine to type ReplicatedMergeTree
```python
gigapipe_client.clusters.engine_to_replicated_merge_tree(
    "cluster-test", 
    table_name="clickhouse_table", 
    payload={
        "partition_by": "my_column_name, my_other_column_name",
        "order_by": "my_column_name, my_other_column_name",
        "database_name": "my_database_name",
        "path": "path_to_new_engine" # Optional: If no path is chosen, a default path is defined automatically
    })

# Payload response
{
    "message": "Table engine changed to replicated merge tree: <>"
}
```

- Import data from an external cluster, creating tables if necessary
> Given a cluster slug, external connection parameters and a list of tables, import data from an external cluster.
```python
gigapipe_client.clusters.import_from_external_cluster(
    "cluster-test",
    external_cluster_params={
        "host": "route_to_host",
        "port": "port number", 
        "database": "database_name",
        "username": "username",
        "password": "password"
    },
    table_arrays=[{
        "table_name": "string",
        "create_table": True
    }]
)

# Payload response
{
    "message": "Successfully imported external data: <>"
}
```

### Clickhouse Elements

#### Use your Gigapipe instance to make calls to the library.

- Create a Clickhouse user
> Creates a user on Clickhouse for a specific cluster
```python
gigapipe_client.clickhouse.create_user("cluster-test", user={
    "username": "Kelly",
    "password": "kelly-pw"
})

# Payload response
{
    "message": "User Kelly created."
}
```

- Get Clickhouse user
> Obtains a Clickhouse user for a specific cluster
```python
gigapipe_client.clickhouse.get_users(cluster_slug="cluster-test")

# Payload response
[
    {
        "name": "Kelly", 
        "id": "2e4b1c8d-5055-02b1-8a4a-0b8b7e274b96", 
        "host_ip": ["::/0"], 
        "host_names": [], 
        "host_names_regexp": [], 
        "host_names_like": [], 
        "default_roles_all": True, 
        "default_roles_list": [], 
        "default_roles_except": [], 
        "grantees_any": True, 
        "grantees_list": [], 
        "grantees_except": []
    }, 
    ...
]
```

- Update a Clickhouse user
> Updates a user on Clickhouse for a specific cluster
```python
gigapipe_client.clickhouse.update_user("cluster-test", user={
    "rename": {
        "username": "Kelly",
        "to": "Maria"
    },
    "change_password": {
        "username": "Maria",
        "password": "maria-pw"
    }
})

# Payload response
{
    "message": "User Kelly updated."
}
```

- Delete a Clickhouse user
> Deletes a user on Clickhouse for a specific cluster
```python
gigapipe_client.clickhouse.delete_user(cluster_slug="cluster-test", username="Maria")

# Payload response
{
    "message": "User Maria deleted."
}
```

- Explore Clickhouse tables
> Describes the clickhouse tables.
> 
> Note that "table" and "engine" are both optional parameters that are especially helpful when wanting to focus the query. When generalizing, we might as well not use them.
>
> This examples also assumes there's a table named "downloads" which has date, user_id and bytes as fields.
```python
gigapipe_client.clickhouse.explore_tables(
    "cluster-test", 
    table_name="downloads",     # Optional
    engine="Distributed"        # Optional
)

# Payload response
{
    "database": "default", 
    "name": "downloads", 
    "engine": "Distributed", 
    "columns": [
        {
            "name": "date", 
            "type": "DateTime"
        }, 
        {
            "name": "user_id", 
            "type": "UInt32"
        }, 
        {
            "name": "bytes", 
            "type": "Float64"
        }
    ], 
    "rows": 50330160
}
```

- Get Formats
> Obtains the clickhouse formats
```python
gigapipe_client.clickhouse.get_formats()

# Payload response
[
    {
        "name": "TabSeparated", 
        "imports": True, 
        "exports": True
    }, 
    {
        "name": "TabSeparatedRaw", 
        "imports": True, 
        "exports": True
    },
    ...
]
```

- Get Versions
> Obtains the clickhouse versions
```python
gigapipe_client.clickhouse.get_versions()

# Payload response
[
  {
    "name": "string",
    "id": 0
  }
]
```

### Imports

#### Use your Gigapipe instance to make calls to the library.

- Import a s3 file
> Imports S3 data
```python
gigapipe_client.imports.import_s3_data("cluster-test", {
    "table": "Your table name",
    "path": "your_s3_path",
    "aws_access_key_id": "your_key_id",
    "aws_secret_access_key": "your_secret_key",
    "format": "CSVWithNames",   # See the formats method to list all the possibilities
    "columns": [
        {
            "name": "Your attribute",
            "type": "UInt64"
        },
        ...
    ],
    "compression": "gzip"
})

# Payload response
{
    "message": "S3 file will be imported."
}
```

- Get the imports
> Obtains all the imports for a given cluster
```python
gigapipe_client.imports.get_imports(cluster_slug="cluster-test")

# Payload response
[
    {
        "table": "Your table name", 
        "source": "AWS S3", 
        "path": "your_s3_path", 
        "format": {
            "name": "CSVWithNames", 
            "imports": True, 
            "exports": True
        }, 
        "status": "In progress", 
        "rows": None, 
        "bytes": None, 
        "duration_ms": None, 
        "error": None, 
        "created_at": "2022-02-16T11:28:21.559138"
    }
    ...
]

# Payload after if has finished importing
[
    {
        "table": "Your table name", 
        "source": "AWS S3", 
        "path": "your_s3_path", 
        "format": {
            "name": "CSVWithNames", 
            "imports": True, 
            "exports": True
        }, 
        "status": "Succeeded", 
        "rows": 122488236, 
        "bytes": 7104317688, 
        "duration_ms": 236832, 
        "error": "", 
        "created_at": "2022-02-16T11:28:21.559138"
    }
]
```

### Integrations

#### Use your Gigapipe instance to make calls to the library.

- Get the integration types
> Obtains the gigapipe integration types
```python
gigapipe_client.integrations.get_integrations_types()

# Payload response
[
    {
        "name": "Kafka", 
        "slug": "kafka", 
        "description": "Apache Kafka is an open-source distributed ..."
    }
    ...
]

```

- Get the integrations
> Obtains the gigapipe integrations
```python
gigapipe_client.integrations.get_integrations()

# Payload response
[
    {
        "name": "Kafka",
        "slug": "kafka",
        "type": {
            "name": "Kafka",
            "slug": "kafka",
            "description": "Apache Kafka is an open-source distributed ..."
        },
        "created_at": "2022-02-17T14:16:07.791Z"
    }
    ...
]
```

### Stripe

#### Use your Gigapipe instance to make calls to the library.

- Post company TAX ID
> Adds or update the tax ID 
```python
gigapipe_client.stripe.post_tax_id({
    'tax_type': 'au_abn',
    'tax_value': '12345678912'
})

# Payload response
{
  "message": "Tax ID updated in organization <organization-slug>"
}

```

- Get the company TAX ID
> Obtains the company TAX ID
```python
gigapipe_client.stripe.get_tax_id()

# Payload response
{
    "tax_type": "au_abn",
    "tax_value": "12345678912",
    "tax_country": "AU"
}
```

- Delete the company TAX ID
> Does away with the company TAX ID
```python
gigapipe_client.stripe.delete_tax_id()

# Payload response
{
    "message": "Tax ID deleted in organization <gigapipe>"
}
```

### Backups

#### Use your Gigapipe instance to make calls to the library.

- Get the organization backups
> Obtains the backups for the clusters belonging to a specific organization
```python
gigapipe_client.backups.get_organization_backups()

# Payload response
[
    {
        "id": 1,
        "timestamp": "1655991226",
        "cronjob": {
            "interval": "* * */5 *",
            "target_metadata": "example",
            "name": "cronjob_name",
            "cluster": {
                "name": "cluster_name",
                "slug": "cluster-slug"
            }
        }
    }
    ...
]
```

- Get the cluster backups
> Obtains the backups for a specific cluster
```python
gigapipe_client.backups.get_cluster_backups('cluster-slug')

# Payload response
[
    {
        "id": 1,
        "timestamp": "1655991226",
        "cronjob": {
            "interval": "* * */5 *",
            "target_metadata": "example",
            "name": "cronjob_name",
            "cluster": {
                "name": "cluster_name",
                "slug": "cluster-slug"
            }
        }
    }
    ...
]
```

- Restore from backup
> Restores a backup in a specific cluster
```python
gigapipe_client.backups.restore_backup('cluster-slug', backup_payload={
    'backup_id': 1
})

# Payload response
{
    "message": "Restore process in progress."
}
```

- Get the organization cronjobs
> Obtains the backup cronjobs belonging to the clusters of the organization
```python
gigapipe_client.backups.get_organization_backups_cronjobs()

# Payload response
[
    {
        "interval": "* * */5 *",
        "target_metadata": "example",
        "name": "cronjob name",
        "id": 1,
        "status": "Active",
        "cluster": {
            "name": "cluster_name",
            "slug": "cluster-slug"
        }
    }
]
```

- Get the cluster cronjobs
> Obtains the backup cronjobs belonging to a specific cluster
```python
gigapipe_client.backups.get_cluster_backups_cronjobs('cluster-slug')

# Payload response
[
    {
        "interval": "* * */5 *",
        "target_metadata": "example",
        "name": "cronjob name",
        "id": 1,
        "status": "Active",
        "cluster": {
            "name": "cluster_name",
            "slug": "cluster-slug"
        }
    }
    ...
]
```

- Get backup cronjob
> Given the ID, it obtains the backup cronjob
```python
gigapipe_client.backups.get_cluster_backups_cronjobs(cronjob_id=1)

# Payload response
[
    {
        "interval": "* * */5 *",
        "target_metadata": "example",
        "name": "cronjob name",
        "id": 1,
        "status": "Active",
        "cluster": {
            "name": "cluster_name",
            "slug": "cluster-slug"
        }
    }
    ...
]
```

- Create a backup cronjob
> Creates a backup cronjob
```python
gigapipe_client.backups.create_backup_cronjob('cluster-slug', cronjob_payload={
    "interval": "* * */5 *",
    "target_metadata": "example",
    "name": "cronjob name"
})

# Payload response
{
    "message": "Backup cronjob creation in progress."
}
```

- Updates a backup cronjob
> Updates a backup cronjob
```python
gigapipe_client.backups.update_backup_cronjob('cluster-slug', cronjob_payload={
    "id": 1,
    "interval": "* * */5 *",
    "target_metadata": "example",
    "name": "cronjob name"
})

# Payload response
{
    "message": "Backup cronjob update in progress."
}
```

- Deletes a backup cronjob
> Given a cluster slug and a cronjob id, it removes a backup cronjob
```python
gigapipe_client.backups.update_backup_cronjob('cluster-slug', cronjob_id=1)

# Payload response
{
    "message": "Backup cronjob delete in progress."
}
```

--- 

_Should you require further information, do not hesitate to visit the [Gigapipe API Docs](https://docs.gigapipe.com/api/introduction)._