# 使用 Operators（执行器）

> 贡献者：[@ImPerat0R\_](https://github.com/tssujt)、[@ThinkingChen](https://github.com/cdmikechen) [@zhongjiajie](https://github.com/zhongjiajie)

operator（执行器）代表一个理想情况下是幂等的任务。operator（执行器）决定了 DAG 运行时实际执行的内容。

有关更多信息，请参阅[Operators Concepts](zh/concepts.md)文档和[Operators API Reference](zh/code.md) 。

* [BashOperator](zh/howto/operator.md)
  * [模板](zh/howto/operator.md)
  * [故障排除](zh/howto/operator.md)
    * [找不到 Jinja 模板](zh/howto/operator.md)
* [PythonOperator](zh/howto/operator.md)
  * [传递参数](zh/howto/operator.md)
  * [模板](zh/howto/operator.md)
* [Google Cloud Platform Operators](zh/howto/operator.md)
  * [GoogleCloudStorageToBigQueryOperator](zh/howto/operator.md)
  * [GceInstanceStartOperator](zh/howto/operator.md)
  * [GceInstanceStopOperator](zh/howto/operator.md)
  * [GceSetMachineTypeOperator](zh/howto/operator.md)
  * [GcfFunctionDeleteOperator](zh/howto/operator.md)
    * [故障排除](zh/howto/operator.md)
  * [GcfFunctionDeployOperator](zh/howto/operator.md)
    * [故障排除](zh/howto/operator.md)
  * [CloudSqlInstanceDatabaseCreateOperator](zh/howto/operator.md)
    * [参数](zh/howto/operator.md)
    * [使用执行器](zh/howto/operator.md)
    * [模版](zh/howto/operator.md)
    * [更多信息](zh/howto/operator.md)
  * [CloudSqlInstanceDatabaseDeleteOperator](zh/howto/operator.md)
    * [参数](zh/howto/operator.md)
    * [使用执行器](zh/howto/operator.md)
    * [模版](zh/howto/operator.md)
    * [更多信息](zh/howto/operator.md)
  * [CloudSqlInstanceDatabasePatchOperator](zh/howto/operator.md)
    * [参数](zh/howto/operator.md)
    * [使用执行器](zh/howto/operator.md)
    * [模版](zh/howto/operator.md)
    * [更多信息](zh/howto/operator.md)
  * [CloudSqlInstanceDeleteOperator](zh/howto/operator.md)
    * [参数](zh/howto/operator.md)
    * [使用执行器](zh/howto/operator.md)
    * [模版](zh/howto/operator.md)
    * [更多信息](zh/howto/operator.md)
  * [CloudSqlInstanceCreateOperator](zh/howto/operator.md)
    * [参数](zh/howto/operator.md)
    * [使用执行器](zh/howto/operator.md)
    * [模版](zh/howto/operator.md)
    * [更多信息](zh/howto/operator.md)
  * [CloudSqlInstancePatchOperator](zh/howto/operator.md)
    * [参数](zh/howto/operator.md)
    * [使用执行器](zh/howto/operator.md)
    * [模版](zh/howto/operator.md)
    * [更多信息](zh/howto/operator.md)

## BashOperator

使用[`BashOperator`](zh/code.md)在[Bash](https://www.gnu.org/software/bash/) shell 中执行命令。

```py
run_this = BashOperator(
    task_id='run_after_loop',
    bash_command='echo 1',
    dag=dag)
```

### 模板

您可以使用[Jinja 模板](zh/concepts.md)来参数化`bash_command`参数。

```py
also_run_this = BashOperator(
    task_id='also_run_this',
    bash_command='echo "run_id={{ run_id }} | dag_run={{ dag_run }}"',
    dag=dag,
)
```

### 故障排除

#### 找不到 Jinja 模板

在使用`bash_command`参数直接调用 Bash 脚本时，需要在脚本名称后添加空格。这是因为 Airflow 尝试将 Jinja 模板应用于一个失败的脚本。

```py
t2 = BashOperator(
    task_id='bash_example',

    # 这将会出现`Jinja template not found`的错误
    # bash_command="/home/batcher/test.sh",

    # 在加了空格之后，这会正常工作
    bash_command="/home/batcher/test.sh ",
    dag=dag)
```

## PythonOperator

使用[`PythonOperator`](zh/code.md)执行 Python 回调。

```py
 def print_context ( ds , ** kwargs ):
    pprint ( kwargs )
    print ( ds )
    return 'Whatever you return gets printed in the logs'

run_this = PythonOperator (
    task_id = 'print_the_context' ,
    provide_context = True ,
    python_callable = print_context ,
    dag = dag )

```

### 传递参数

使用`op_args`和`op_kwargs`参数将额外参数传递给 Python 的回调函数。

```py
def my_sleeping_function(random_base):
    """这是一个将在 DAG 执行体中运行的函数"""
    time.sleep(random_base)


# Generate 10 sleeping tasks, sleeping from 0 to 4 seconds respectively
for i in range(5):
    task = PythonOperator(
        task_id='sleep_for_' + str(i),
        python_callable=my_sleeping_function,
        op_kwargs={'random_base': float(i) / 10},
        dag=dag,
    )

    run_this >> task
```

### 模板

当您将`provide_context`参数设置为`True`，Airflow 会传入一组额外的关键字参数：一个用于每个[Jinja 模板变量](zh/code.md)和一个`templates_dict`参数。

`templates_dict`参数是模板化的，因此字典中的每个值都被评估为[Jinja 模板](zh/howto/operator.md)。

## Google 云平台 Operators（执行器）

### GoogleCloudStorageToBigQueryOperator

使用[`GoogleCloudStorageToBigQueryOperator`](zh/integration.md)执行 BigQuery 加载作业。

### GceInstanceStartOperator

允许启动一个已存在的 Google Compute Engine 实例。

在此示例中，参数值从 Airflow 变量中提取。此外，`default_args`字典用于将公共参数传递给单个 DAG 中的所有 operator（执行器）。

```py
PROJECT_ID = models.Variable.get('PROJECT_ID', '')
LOCATION = models.Variable.get('LOCATION', '')
INSTANCE = models.Variable.get('INSTANCE', '')
SHORT_MACHINE_TYPE_NAME = models.Variable.get('SHORT_MACHINE_TYPE_NAME', '')
SET_MACHINE_TYPE_BODY = {
    'machineType': 'zones/{}/machineTypes/{}'.format(LOCATION, SHORT_MACHINE_TYPE_NAME)
}

default_args = {
    'start_date': airflow.utils.dates.days_ago(1)
}
```

通过将所需的参数传递给构造函数来定义`GceInstanceStartOperator`。

```py
gce_instance_start = GceInstanceStartOperator(
    project_id=PROJECT_ID,
    zone=LOCATION,
    resource_id=INSTANCE,
    task_id='gcp_compute_start_task'
)
```

### GceInstanceStopOperator

允许停止一个已存在的 Google Compute Engine 实例。

参数定义请参阅上面的`GceInstanceStartOperator`。

通过将所需的参数传递给构造函数来定义`GceInstanceStopOperator`。

```py
gce_instance_stop = GceInstanceStopOperator(
    project_id=PROJECT_ID,
    zone=LOCATION,
    resource_id=INSTANCE,
    task_id='gcp_compute_stop_task'
)
```

### GceSetMachineTypeOperator

允许把一个已停止实例的机器类型改变至特定的类型。

参数定义请参阅上面的`GceInstanceStartOperator`。

通过将所需的参数传递给构造函数来定义`GceSetMachineTypeOperator`。

```py
gce_set_machine_type = GceSetMachineTypeOperator(
    project_id=PROJECT_ID,
    zone=LOCATION,
    resource_id=INSTANCE,
    body=SET_MACHINE_TYPE_BODY,
    task_id='gcp_compute_set_machine_type'
)
```

### GcfFunctionDeleteOperator

使用`default_args`字典来传递参数给 operator（执行器）。

```py
PROJECT_ID = models.Variable.get('PROJECT_ID', '')
LOCATION = models.Variable.get('LOCATION', '')
ENTRYPOINT = models.Variable.get('ENTRYPOINT', '')
# A fully-qualified name of the function to delete

FUNCTION_NAME = 'projects/{}/locations/{}/functions/{}'.format(PROJECT_ID, LOCATION,
                                                               ENTRYPOINT)
default_args = {
    'start_date': airflow.utils.dates.days_ago(1)
}
```

使用`GcfFunctionDeleteOperator`来从 Google Cloud Functions 删除一个函数。

```py
t1 = GcfFunctionDeleteOperator(
    task_id="gcf_delete_task",
    name=FUNCTION_NAME
)
```

#### 故障排除

如果你想要使用服务账号来运行或部署一个 operator（执行器），但得到了一个 403 禁止的错误，这意味着你的服务账号没有正确的 Cloud IAM 权限。

1. 指定该服务账号为 Cloud Functions Developer 角色。
2. 授权 Cloud Functions 的运行账户为 Cloud IAM Service Account User 角色。

使用 gcloud 分配 Cloud IAM 权限的典型方法如下所示。只需将您的 Google Cloud Platform 项目 ID 替换为 PROJECT_ID，将 SERVICE_ACCOUNT_EMAIL 替换为您的服务帐户的电子邮件 ID 即可。

```py
gcloud iam service-accounts add-iam-policy-binding \
  PROJECT_ID@appspot.gserviceaccount.com \
  --member="serviceAccount:[SERVICE_ACCOUNT_EMAIL]" \
  --role="roles/iam.serviceAccountUser"
```

细节请参阅[Adding the IAM service agent user role to the runtime service](https://cloud.google.com/functions/docs/reference/iam/roles#adding_the_iam_service_agent_user_role_to_the_runtime_service_account)

### GcfFunctionDeployOperator

使用`GcfFunctionDeployOperator`来从 Google Cloud Functions 部署一个函数。

以下 Airflow 变量示例显示了您可以使用的 default_args 的各种变体和组合。变量定义如下：

```py
PROJECT_ID = models.Variable.get('PROJECT_ID', '')
LOCATION = models.Variable.get('LOCATION', '')
SOURCE_ARCHIVE_URL = models.Variable.get('SOURCE_ARCHIVE_URL', '')
SOURCE_UPLOAD_URL = models.Variable.get('SOURCE_UPLOAD_URL', '')
SOURCE_REPOSITORY = models.Variable.get('SOURCE_REPOSITORY', '')
ZIP_PATH = models.Variable.get('ZIP_PATH', '')
ENTRYPOINT = models.Variable.get('ENTRYPOINT', '')
FUNCTION_NAME = 'projects/{}/locations/{}/functions/{}'.format(PROJECT_ID, LOCATION,
                                                               ENTRYPOINT)
RUNTIME = 'nodejs6'
VALIDATE_BODY = models.Variable.get('VALIDATE_BODY', True)
```

使用这些变量，您可以定义请求的主体：

```py
body = {
    "name": FUNCTION_NAME,
    "entryPoint": ENTRYPOINT,
    "runtime": RUNTIME,
    "httpsTrigger": {}
}
```
创建 DAG 时，default_args 字典可用于传递正文和其他参数：

```py
default_args = {
    'start_date': dates.days_ago(1),
    'project_id': PROJECT_ID,
    'location': LOCATION,
    'body': body,
    'validate_body': VALIDATE_BODY
}
```
请注意，在上面的示例中，body 和 default_args 都是不完整的。根据设置的变量，如何传递源代码相关字段可能有不同的变体。目前，您可以传递 sourceArchiveUrl，sourceRepository 或 sourceUploadUrl，[CloudFunction API 规范](https://cloud.google.com/functions/docs/reference/rest/v1/projects.locations.functions#CloudFunction)中所述。此外，default_args 可能包含 zip_path 参数，以在部署源代码之前运行上载源代码的额外步骤。在最后一种情况下，您还需要在正文中提供一个空的 sourceUploadUrl 参数。

基于上面定义的变量，此处显示了设置源代码相关字段的示例逻辑：

```py
if SOURCE_ARCHIVE_URL:
    body['sourceArchiveUrl'] = SOURCE_ARCHIVE_URL
elif SOURCE_REPOSITORY:
    body['sourceRepository'] = {
        'url': SOURCE_REPOSITORY
    }
elif ZIP_PATH:
    body['sourceUploadUrl'] = ''
    default_args['zip_path'] = ZIP_PATH
elif SOURCE_UPLOAD_URL:
    body['sourceUploadUrl'] = SOURCE_UPLOAD_URL
else:
    raise Exception("Please provide one of the source_code parameters")
```

创建 operator（执行器）的代码如下：

```py
deploy_task = GcfFunctionDeployOperator(
    task_id="gcf_deploy_task",
    name=FUNCTION_NAME
)
```

#### Troubleshooting

如果你想要使用服务账号来运行或部署一个 operator（执行器），但得到了一个 403 禁止的错误，这意味着你的服务账号没有正确的 Cloud IAM 权限。

1. 指定该服务账号为 Cloud Functions Developer 角色。
2. 授权 Cloud Functions 的运行账户为 Cloud IAM Service Account User 角色。

使用 gcloud 分配 Cloud IAM 权限的典型方法如下所示。只需将您的 Google Cloud Platform 项目 ID 替换为 PROJECT_ID，将 SERVICE_ACCOUNT_EMAIL 的替换为您的服务帐户的电子邮件 ID 即可。

```py
gcloud iam service-accounts add-iam-policy-binding \
  PROJECT_ID@appspot.gserviceaccount.com \
  --member="serviceAccount:[SERVICE_ACCOUNT_EMAIL]" \
  --role="roles/iam.serviceAccountUser"
```

细节请参阅[Adding the IAM service agent user role to the runtime service](https://cloud.google.com/functions/docs/reference/iam/roles#adding_the_iam_service_agent_user_role_to_the_runtime_service_account)

如果您的函数的源代码位于 Google Source Repository 中，请确保您的服务帐户具有 Source Repository Viewer 角色，以便在必要时可以下载源代码。

### CloudSqlInstanceDatabaseCreateOperator

在 Cloud SQL 实例中创建新数据库。

有关参数定义，请参阅上面的`GceInstanceStartOperator`。

通过将所需的参数传递给构造函数来定义`CloudSqlInstanceDatabaseCreateOperator`。

#### 参数

示例 DAG 中的一些参数取自环境变量：

```py
PROJECT_ID = os.environ.get('PROJECT_ID', 'example-project')
INSTANCE_NAME = os.environ.get('INSTANCE_NAME', 'testinstance')
DB_NAME = os.environ.get('DB_NAME', 'testdb')
```

#### 使用 operator（执行器）

```py
sql_db_create_task = CloudSqlInstanceDatabaseCreateOperator(
    project_id=PROJECT_ID,
    body=db_create_body,
    instance=INSTANCE_NAME,
    task_id='sql_db_create_task'
)
```

示例请求体：

```py
db_create_body = {
    "instance": INSTANCE_NAME,
    "name": DB_NAME,
    "project": PROJECT_ID
}
```

#### 模版

```py
template_fields = ('project_id', 'instance', 'gcp_conn_id', 'api_version')
```

#### 更多信息

有关数据库插入，请参阅[Google Cloud SQL API 文档](https://cloud.google.com/sql/docs/mysql/admin-api/v1beta4/databases/insert)。

### CloudSqlInstanceDatabaseDeleteOperator

在 Cloud SQL 实例中删除数据库。

有关参数定义，请参阅`CloudSqlInstanceDatabaseDeleteOperator`。

#### 参数

示例 DAG 中的一些参数取自环境变量：

```py
PROJECT_ID = os.environ.get('PROJECT_ID', 'example-project')
INSTANCE_NAME = os.environ.get('INSTANCE_NAME', 'testinstance')
DB_NAME = os.environ.get('DB_NAME', 'testdb')
```

#### 使用 operator（执行器）

```py
sql_db_delete_task = CloudSqlInstanceDatabaseDeleteOperator(
    project_id=PROJECT_ID,
    instance=INSTANCE_NAME,
    database=DB_NAME,
    task_id='sql_db_delete_task'
)
```

#### 模版

```py
template_fields = ('project_id', 'instance', 'database', 'gcp_conn_id',
                   'api_version')
```

#### 更多信息

有关数据库删除，请参阅[Google Cloud SQL API 文档](https://cloud.google.com/sql/docs/mysql/admin-api/v1beta4/databases/delete)。

### CloudSqlInstanceDatabasePatchOperator

使用修补程序语义更新包含有关 Cloud SQL 实例内数据库的信息的资源。请参阅: https://cloud.google.com/sql/docs/mysql/admin-api/how-tos/performance#patch

有关参数定义，请参阅`CloudSqlInstanceDatabasePatchOperator`。

#### 参数

示例 DAG 中的一些参数取自环境变量：

```py
PROJECT_ID = os.environ.get('PROJECT_ID', 'example-project')
INSTANCE_NAME = os.environ.get('INSTANCE_NAME', 'testinstance')
DB_NAME = os.environ.get('DB_NAME', 'testdb')
```

#### 使用 operator（执行器）

```py
sql_db_patch_task = CloudSqlInstanceDatabasePatchOperator(
    project_id=PROJECT_ID,
    body=db_patch_body,
    instance=INSTANCE_NAME,
    database=DB_NAME,
    task_id='sql_db_patch_task'
)
```

示例请求体：

```py
db_patch_body = {
    "charset": "utf16",
    "collation": "utf16_general_ci"
}
```

#### 模版
```py
template_fields = ('project_id', 'instance', 'database', 'gcp_conn_id',
                   'api_version')
```

#### 更多信息

有关数据库修改，请参阅[Google Cloud SQL API 文档](https://cloud.google.com/sql/docs/mysql/admin-api/v1beta4/databases/patch)。

### CloudSqlInstanceDeleteOperator

示例 DAG 中的一些参数取自环境变量：

```py
PROJECT_ID = os.environ.get('PROJECT_ID', 'example-project')
INSTANCE_NAME = os.environ.get('INSTANCE_NAME', 'testinstance')
DB_NAME = os.environ.get('DB_NAME', 'testdb')
```

#### 使用 operator（执行器）

```py
sql_instance_delete_task = CloudSqlInstanceDeleteOperator(
    project_id=PROJECT_ID,
    instance=INSTANCE_NAME,
    task_id='sql_instance_delete_task'
)
```

#### 模版
```py
template_fields = ('project_id', 'instance', 'gcp_conn_id', 'api_version')
```

#### 更多信息

有关删除，请参阅[Google Cloud SQL API 文档](https://cloud.google.com/sql/docs/mysql/admin-api/v1beta4/instances/delete)。

### CloudSqlInstanceCreateOperator

在 Google Cloud Platform 中创建新的 Cloud SQL 实例。

有关参数定义，请参阅`CloudSqlInstanceCreateOperator`。

如果存在具有相同名称的实例，则不会执行任何操作，并且 operator（执行器）将成功执行。

#### 参数

示例 DAG 中的一些参数取自环境变量：

```py
PROJECT_ID = os.environ.get('PROJECT_ID', 'example-project')
INSTANCE_NAME = os.environ.get('INSTANCE_NAME', 'testinstance')
DB_NAME = os.environ.get('DB_NAME', 'testdb')
```

定义实例的示例：
```py
body = {
    "name": INSTANCE_NAME,
    "settings": {
        "tier": "db-n1-standard-1",
        "backupConfiguration": {
            "binaryLogEnabled": True,
            "enabled": True,
            "startTime": "05:00"
        },
        "activationPolicy": "ALWAYS",
        "dataDiskSizeGb": 30,
        "dataDiskType": "PD_SSD",
        "databaseFlags": [],
        "ipConfiguration": {
            "ipv4Enabled": True,
            "requireSsl": True,
        },
        "locationPreference": {
            "zone": "europe-west4-a"
        },
        "maintenanceWindow": {
            "hour": 5,
            "day": 7,
            "updateTrack": "canary"
        },
        "pricingPlan": "PER_USE",
        "replicationType": "ASYNCHRONOUS",
        "storageAutoResize": False,
        "storageAutoResizeLimit": 0,
        "userLabels": {
            "my-key": "my-value"
        }
    },
    "databaseVersion": "MYSQL_5_7",
    "region": "europe-west4",
}
```

#### 使用 operator（执行器）

```py
sql_instance_create_task = CloudSqlInstanceCreateOperator(
    project_id=PROJECT_ID,
    body=body,
    instance=INSTANCE_NAME,
    task_id='sql_instance_create_task'
)
```

#### 模版
```py
template_fields = ('project_id', 'instance', 'gcp_conn_id', 'api_version')
```

#### 更多信息

有关插入，请参阅[Google Cloud SQL API 文档](https://cloud.google.com/sql/docs/mysql/admin-api/v1beta4/instances/insert)。

### CloudSqlInstancePatchOperator

更新 Google Cloud Platform 中的 Cloud SQL 实例的设置（部分更新）。

有关参数定义，请参阅`CloudSqlInstancePatchOperator`。

这是部分更新，因此仅设置/更新正文中指定的设置的值。现有实例的其余部分将保持不变。

#### 参数

示例 DAG 中的一些参数取自环境变量：

```py
PROJECT_ID = os.environ.get('PROJECT_ID', 'example-project')
INSTANCE_NAME = os.environ.get('INSTANCE_NAME', 'testinstance')
DB_NAME = os.environ.get('DB_NAME', 'testdb')
```

定义实例的示例：
```py
patch_body = {
    "name": INSTANCE_NAME,
    "settings": {
        "dataDiskSizeGb": 35,
        "maintenanceWindow": {
            "hour": 3,
            "day": 6,
            "updateTrack": "canary"
        },
        "userLabels": {
            "my-key-patch": "my-value-patch"
        }
    }
}
```

#### 使用 operator（执行器）

```py
sql_instance_patch_task = CloudSqlInstancePatchOperator(
    project_id=PROJECT_ID,
    body=patch_body,
    instance=INSTANCE_NAME,
    task_id='sql_instance_patch_task'
)
```

#### 模版
```py
template_fields = ('project_id', 'instance', 'gcp_conn_id', 'api_version')
```

#### 更多信息

有关部分更新，请参阅[Google Cloud SQL API 文档](https://cloud.google.com/sql/docs/mysql/admin-api/v1beta4/instances/patch)。
