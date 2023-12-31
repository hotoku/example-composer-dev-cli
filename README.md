# 動作確認

## 環境を作成

```shell
$ composer-dev create \
  --from-image-version composer-2.5.1-airflow-2.6.3 \
  example
```

↑最低限の指定: その他のオプションはマニュアル参照

- image-versionごとに、imageファイルを保存する。不要なバージョンで作るとディスクを食う: 5.4GB

## 起動

```shell
$ composer-dev start example
```

- web uiのport
- dagsディレクトリの場所を出力から確認

デフォルトでは[http://localhost:8080](http://localhost:8080)でweb uiが起動

## dag登録

以下を、dagsの下に`tutorial.py`で保存。web uiをreloadでdagが登録されているのを確認。

```python

from datetime import datetime, timedelta
from textwrap import dedent

# The DAG object; we'll need this to instantiate a DAG
from airflow import DAG

# Operators; we need this to operate!
from airflow.operators.bash import BashOperator
with DAG(
    "tutorial",
    # These args will get passed on to each operator
    # You can override them on a per-task basis during operator initialization
    default_args={
        "depends_on_past": False,
        "email": ["airflow@example.com"],
        "email_on_failure": False,
        "email_on_retry": False,
        "retries": 1,
        "retry_delay": timedelta(minutes=5),
        # 'queue': 'bash_queue',
        # 'pool': 'backfill',
        # 'priority_weight': 10,
        # 'end_date': datetime(2016, 1, 1),
        # 'wait_for_downstream': False,
        # 'sla': timedelta(hours=2),
        # 'execution_timeout': timedelta(seconds=300),
        # 'on_failure_callback': some_function, # or list of functions
        # 'on_success_callback': some_other_function, # or list of functions
        # 'on_retry_callback': another_function, # or list of functions
        # 'sla_miss_callback': yet_another_function, # or list of functions
        # 'trigger_rule': 'all_success'
    },
    description="A simple tutorial DAG",
    schedule=timedelta(days=1),
    start_date=datetime(2021, 1, 1),
    catchup=False,
    tags=["example"],
) as dag:

    # t1, t2 and t3 are examples of tasks created by instantiating operators
    t1 = BashOperator(
        task_id="print_date",
        bash_command="date",
    )

    t2 = BashOperator(
        task_id="sleep",
        depends_on_past=False,
        bash_command="sleep 5",
        retries=3,
    )
    t1.doc_md = dedent(
        """\
    #### Task Documentation
    You can document your task using the attributes `doc_md` (markdown),
    `doc` (plain text), `doc_rst`, `doc_json`, `doc_yaml` which gets
    rendered in the UI's Task Instance Details page.
    ![img](http://montcs.bloomu.edu/~bobmon/Semesters/2012-01/491/import%20soul.png)
    **Image Credit:** Randall Munroe, [XKCD](https://xkcd.com/license.html)
    """
    )

    dag.doc_md = __doc__  # providing that you have a docstring at the beginning of the DAG; OR
    dag.doc_md = """
    This is a documentation placed anywhere
    """  # otherwise, type it like this
    templated_command = dedent(
        """
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7)}}"
    {% endfor %}
    """
    )

    t3 = BashOperator(
        task_id="templated",
        depends_on_past=False,
        bash_command=templated_command,
    )

    t1 >> [t2, t3]
```

## 実行

web uiの再生ボタン → 「Trigger DAG」

## Airflowコマンドの実行

```shell
composer-dev run-airflow-cmd example <subcommand> <argument>
```

が基本形

`composer dev run-airflow-cmd example` ...

- `help` ヘルプ表示
- `dags list` ダグ一覧
- `dags trigger <ダグ名>` ダグ実行

## 既存のcomposerから作成

```shell
composer-dev create example2 \
    --from-source-environment pubtex-ai-mlops-job \
    --location asia-northeast1 \
    --project pubtex-ai-trial-dev-mlops-d2
```

全ての情報がコピーされる訳ではなく、以下のみがコピーされる。

- イメージのバージョン（環境で使用されている Cloud Composer と Airflow のバージョン）。
- 使用する環境にインストールされているカスタム PyPI パッケージのリスト。
- 使用する環境に設定された環境変数の名前のコメント化されたリスト。

結果、上記のコマンドでは、ほぼ空の設定がコピーされた。
