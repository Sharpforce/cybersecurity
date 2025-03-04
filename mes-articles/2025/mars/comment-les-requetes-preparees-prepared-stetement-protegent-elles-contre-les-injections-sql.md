---
description: 04 mars 2025
---

# Comment les requêtes préparées (prepared stetement) protègent-elles contre les injections SQL ?

Les injections SQL sont des vulnérabilités largement connues et souvent critiques, mais elles demeurent encore très répandues dans les applications web actuelles. La principale protection contre cette vulnérabilité repose sur l'utilisation des requêtes préparées, mais comment fonctionnent-elles exactement ?

> Les exemples de code présentés utilisent PHP et l'extension MySQLi, mais les mêmes principes s'appliquent à d'autres langages.

## Fonctionnement d'une requête directe

### Exemple d'une injection SQL avec query()

Avant de plonger dans le fonctionnement des requêtes préparées, il est essentiel de rappeler ce qu'est une injection SQL et comment elle se produit. Une injection SQL survient lorsqu'une donnée fournie par l'utilisateur (ou toute autre source dite "non fiable") est insérée directement dans une requête SQL, sans validation ni assainissement préalable.

Un exemple de cette vulnérabilité pourrait être une fonctionnalité permettant à l'utilisateur d'accéder aux informations de différentes villes de France. Lorsqu'un utilisateur consulte les informations d'une ville, la requête `GET` suivante est envoyée, précisant le nom de la ville :&#x20;

```http
GET /city?name=paris HTTP/1.1
```

```php
$city_name = $_GET['name'];
$sql = "SELECT name, population FROM cities WHERE name = '$city_name'";

$result = $cnx->query($sql);
```

L'injection est relativement simple. La tautologie permet de récupérer l'ensemble des enregistrements de la table, tandis que l'utilisation de l'`UNION` permet d'extraire des informations provenant d'autres tables :&#x20;

```http
GET /city?name=' OR 1=1# HTTP/1.1

City: Paris
Population: 2148327
City: Marseille
Population: 861635
City: Lyon
Population: 513275
City: Toulouse
Population: 479175
```

```http
GET /city?name=' UNION SELECT username,password FROM users# HTTP/1.1

City: azerty
Population: admin
```

Mais comment, et pourquoi, l'injection SQL se produit-elle réellement dans le code, notamment au niveau de l'extension MySQLi de PHP ?

### Analyse du code PHP lors de l'exécution d'une requête directe

Lors de l'appel à la méthode PHP `query()`, la fonction `mysqli_query()` (qui est un alias défini dans **ext/mysqli/mysqli.stub.php**) présente dans le fichier **ext/mysqli/mysqli\_nonapi.c** est exécutée :&#x20;

```c
PHP_FUNCTION(mysqli_query)
{
    MY_MYSQL		*mysql;
    zval		*mysql_link;
    MYSQLI_RESOURCE	*mysqli_resource;
    MYSQL_RES 		*result = NULL;
    char		*query = NULL;
    size_t 		query_len;
    zend_long 		resultmode = MYSQLI_STORE_RESULT;

    if (zend_parse_method_parameters(ZEND_NUM_ARGS(), getThis(), "Os|l", &mysql_link, mysqli_link_class_entry, &query, &query_len, &resultmode) == FAILURE) {
        RETURN_THROWS();
    }

    if (!query_len) {
        zend_argument_must_not_be_empty_error(ERROR_ARG_POS(2));
        RETURN_THROWS();
    }
    
    if ((resultmode & ~MYSQLI_ASYNC) != MYSQLI_USE_RESULT &&
       MYSQLI_STORE_RESULT != (resultmode & ~(MYSQLI_ASYNC | MYSQLI_STORE_RESULT_COPY_DATA))
       ) {
        zend_argument_value_error(ERROR_ARG_POS(3), "must be either MYSQLI_USE_RESULT or MYSQLI_STORE_RESULT with MYSQLI_ASYNC as an optional bitmask flag");
	RETURN_THROWS();
    }

    MYSQLI_FETCH_RESOURCE_CONN(mysql, mysql_link, MYSQLI_STATUS_VALID);
    MYSQLI_DISABLE_MQ;

    if (resultmode & MYSQLI_ASYNC) {
        if (mysqli_async_query(mysql->mysql, query, query_len)) {
	    MYSQLI_REPORT_MYSQL_ERROR(mysql->mysql);
	    RETURN_FALSE;
	}
        mysql->async_result_fetch_type = resultmode & ~MYSQLI_ASYNC;
	RETURN_TRUE;
    }

    if (mysql_real_query(mysql->mysql, query, query_len)) {
        MYSQLI_REPORT_MYSQL_ERROR(mysql->mysql);
	RETURN_FALSE;
    }

    if (!mysql_field_count(mysql->mysql)) {
        /* no result set - not a SELECT */
	if (MyG(report_mode) & MYSQLI_REPORT_INDEX) {
	    php_mysqli_report_index(query, mysqli_server_status(mysql->mysql));
	}
	RETURN_TRUE;
    }

    switch (resultmode & ~(MYSQLI_ASYNC | MYSQLI_STORE_RESULT_COPY_DATA)) {
        case MYSQLI_STORE_RESULT:
	    result = mysql_store_result(mysql->mysql);
	    break;
	case MYSQLI_USE_RESULT:
	    result = mysql_use_result(mysql->mysql);
	    break;
    }
    
    if (!result) {
        MYSQLI_REPORT_MYSQL_ERROR(mysql->mysql);
	RETURN_FALSE;
    }

    if (MyG(report_mode) & MYSQLI_REPORT_INDEX) {
        php_mysqli_report_index(query, mysqli_server_status(mysql->mysql));
    }

    mysqli_resource = (MYSQLI_RESOURCE *)ecalloc (1, sizeof(MYSQLI_RESOURCE));
    mysqli_resource->ptr = (void *)result;
    mysqli_resource->status = MYSQLI_STATUS_VALID;
    MYSQLI_RETVAL_RESOURCE(mysqli_resource, mysqli_result_class_entry);
}
```

Pour commencer, plusieurs vérifications sont effectuées, notamment celle des arguments passés à la méthode `query()` à l'aide de l'appel à `zend_parse_method_parameters()`, dont le code est présent dans le fichier **Zend/zend\_API.c** :

```c
if (zend_parse_method_parameters(ZEND_NUM_ARGS(), getThis(), "Os|l", &mysql_link, mysqli_link_class_entry, &query, &query_len, &resultmode) == FAILURE) {
    RETURN_THROWS();
}
```

Ensuite, vient une seconde vérification qui consiste à s'assurer que la chaîne de caractères représentant la requête SQL n'est pas vide :&#x20;

```c
if (!query_len) {
  zend_argument_must_not_be_empty_error(ERROR_ARG_POS(2));
  RETURN_THROWS();
}
```

Puis une troisième vérification, concernant le mode de résultat retourné par le serveur MySQL (le second paramètre de la méthode `mysqli::query()` de PHP qui possède la valeur par défaut `MYSQLI_STORE_RESULT`) :&#x20;

```c
 if ((resultmode & ~MYSQLI_ASYNC) != MYSQLI_USE_RESULT &&
     MYSQLI_STORE_RESULT != (resultmode & ~(MYSQLI_ASYNC | MYSQLI_STORE_RESULT_COPY_DATA))
     ) {
     zend_argument_value_error(ERROR_ARG_POS(3), "must be either MYSQLI_USE_RESULT or MYSQLI_STORE_RESULT with MYSQLI_ASYNC as an optional bitmask flag");
     RETURN_THROWS();
 }
```

Puis finalement la requête SQL est exécutée (cas par défaut) via un appel à la méthode `mysql_real_query()` :&#x20;

```c
if (mysql_real_query(mysql->mysql, query, query_len)) {
    MYSQLI_REPORT_MYSQL_ERROR(mysql->mysql);
    RETURN_FALSE;
}
```

{% hint style="info" %}
Le paramètre `query` est la chaîne de caractères (un pointeur plus exactement) représentant la requête SQL.
{% endhint %}

Cette méthode est définie en réalité par la méthode `mysqlnd_query()` ("mysqlnd" pour MySQL Native Driver) via une macro présente dans le fichier **ext/mysqlnd/mysqlnd\_libmysql\_compat.h**) :&#x20;

```c
#define mysql_real_query(r,a,b)    mysqlnd_query((r), (a), (b))
```

Puis par une seconde macro présente dans le fichier **ext/mysqlnd/mysqlnd.h** :&#x20;

```c
#define mysqlnd_query(conn, query_str, query_len)    ((conn)->data)->m->query((conn)->data, (query_str), (query_len))
```

Ce qui conduit à la fonction `query()` présente dans le fichier **ext/mysqlnd/mysqlnd\_connection.c** :&#x20;

```c
static enum_func_status
MYSQLND_METHOD(mysqlnd_conn_data, query)(MYSQLND_CONN_DATA * conn, const char * const query, const size_t query_len)
{
    enum_func_status ret = FAIL;
    DBG_ENTER("mysqlnd_conn_data::query");
    DBG_INF_FMT("conn=%p conn=%" PRIu64 " query=%s", conn, conn->thread_id, query);

    if (PASS == conn->m->send_query(conn, query, query_len, NULL, NULL) && PASS == conn->m->reap_query(conn))
    {
        ret = PASS;
      	if (conn->last_query_type == QUERY_UPSERT && UPSERT_STATUS_GET_AFFECTED_ROWS(conn->upsert_status)) {
	    MYSQLND_INC_CONN_STATISTIC_W_VALUE(conn->stats, STAT_ROWS_AFFECTED_NORMAL, UPSERT_STATUS_GET_AFFECTED_ROWS(conn->upsert_status));
	}
    }
    DBG_RETURN(ret);
}
```

Qui mène à l'exécution de la méthode `send_query()` présente dans le même fichier :&#x20;

```c
static enum_func_status
MYSQLND_METHOD(mysqlnd_conn_data, send_query)(
MYSQLND_CONN_DATA * conn, const char * const query, const size_t query_len,
zval *read_cb, zval *err_cb)
{
    DBG_ENTER("mysqlnd_conn_data::send_query");
    DBG_INF_FMT("conn=%" PRIu64 " query=%s", conn->thread_id, query);
    DBG_INF_FMT("conn->server_status=%u", UPSERT_STATUS_GET_SERVER_STATUS(conn->upsert_status));

    const MYSQLND_CSTRING query_string = {query, query_len};
    enum_func_status ret = conn->command->query(conn, query_string);
    DBG_INF_FMT("conn->server_status=%u", UPSERT_STATUS_GET_SERVER_STATUS(conn->upsert_status));
    DBG_RETURN(ret);
}
```

Puis à l'exécution de la méthode `query()` du fichier **ext/mysqlnd/mysqlnd\_commands.c** :&#x20;

```c
static enum_func_status
MYSQLND_METHOD(mysqlnd_command, query)(MYSQLND_CONN_DATA * const conn, MYSQLND_CSTRING query)
{
    func_mysqlnd_protocol_payload_decoder_factory__send_command send_command = conn->payload_decoder_factory->m.send_command;
    enum_func_status ret = FAIL;

    DBG_ENTER("mysqlnd_command::query");

    ret = send_command(conn->payload_decoder_factory, COM_QUERY, (const zend_uchar*) query.s, query.l, FALSE,
        &conn->state,
	conn->error_info,
	conn->upsert_status,
	conn->stats,
	conn->m->send_close,
	conn);

    if (PASS == ret) {
        SET_CONNECTION_STATE(&conn->state, CONN_QUERY_SENT);
    }

    DBG_RETURN(ret);
}
```

Et finalement, à l'envoi de la commande grâce à la méthode `send_command()` présente dans le fichier **ext/mysqlnd/mysqlnd\_wireprotocol.c**. Cette méthode effectuant un appel à `PACKET_WRITE()` pour transmettre le paquet (la requête SQL) au serveur MySQL :&#x20;

{% hint style="info" %}
A noter que l'argument `command` possède ici la valeur `COM_QUERY`.
{% endhint %}

```c
/* {{{ mysqlnd_protocol::send_command */
static enum_func_status
MYSQLND_METHOD(mysqlnd_protocol, send_command)(
    MYSQLND_PROTOCOL_PAYLOAD_DECODER_FACTORY * payload_decoder_factory,
    const enum php_mysqlnd_server_command command,
    const zend_uchar * const arg, const size_t arg_len,
    const bool silent,

    struct st_mysqlnd_connection_state * connection_state,
    MYSQLND_ERROR_INFO * error_info,
    MYSQLND_UPSERT_STATUS * upsert_status,
    MYSQLND_STATS * stats,
    func_mysqlnd_conn_data__send_close send_close,
    void * send_close_ctx)
{
    enum_func_status ret = PASS;
    MYSQLND_PACKET_COMMAND cmd_packet;
    enum mysqlnd_connection_state state;
    DBG_ENTER("mysqlnd_protocol::send_command");
    DBG_INF_FMT("command=%s silent=%u", mysqlnd_command_to_text[command], silent);
    DBG_INF_FMT("server_status=%u", UPSERT_STATUS_GET_SERVER_STATUS(upsert_status));
    DBG_INF_FMT("sending %zu bytes", arg_len + 1); /* + 1 is for the command */
    state = connection_state->m->get(connection_state);

    switch (state) {
        case CONN_READY:
            break;
        case CONN_QUIT_SENT:
            SET_CLIENT_ERROR(error_info, CR_SERVER_GONE_ERROR, UNKNOWN_SQLSTATE, mysqlnd_server_gone);
            DBG_ERR("Server is gone");
            DBG_RETURN(FAIL);
        default:
            SET_CLIENT_ERROR(error_info, CR_COMMANDS_OUT_OF_SYNC, UNKNOWN_SQLSTATE, mysqlnd_out_of_sync);
            DBG_ERR_FMT("Command out of sync. State=%u", state);
            DBG_RETURN(FAIL);
    }

    UPSERT_STATUS_SET_AFFECTED_ROWS_TO_ERROR(upsert_status);
    SET_EMPTY_ERROR(error_info);

    payload_decoder_factory->m.init_command_packet(&cmd_packet);

    cmd_packet.command = command;
    if (arg && arg_len) {
        cmd_packet.argument.s = (char *) arg;
        cmd_packet.argument.l = arg_len;
    }

    MYSQLND_INC_CONN_STATISTIC(stats, STAT_COM_QUIT + command - 1 /* because of COM_SLEEP */ );

    if (! PACKET_WRITE(payload_decoder_factory->conn, &cmd_packet)) {
        if (!silent && error_info->error_no != CR_SERVER_GONE_ERROR) {
            DBG_ERR_FMT("Error while sending %s packet", mysqlnd_command_to_text[command]);
            php_error(E_WARNING, "Error while sending %s packet. PID=%d", mysqlnd_command_to_text[command], getpid());
        }
        connection_state->m->set(connection_state, CONN_QUIT_SENT);
        send_close(send_close_ctx);
        DBG_ERR("Server is gone");
        ret = FAIL;
    }
    PACKET_FREE(&cmd_packet);
    DBG_RETURN(ret);
}
```

La chaîne de caractères (`const zend_uchar * const arg`) , représentant la requête SQL renseignée par le développeur en tant que paramètre de la méthode `query()`, est envoyée sans aucun traitement particulier en matière de sécurité. Lors de l'exploitation de l'injection par un attaquant, le serveur MySQL recevra ainsi la chaîne de caractères suivante, sans possibilité de distinguer la requête originale des données potentiellement malveillantes fournies par l'attaquant :&#x20;

```sql
SELECT id, name, population FROM cities WHERE name = '' OR 1=1#'
```

Maintenant que le fonctionnement d'une requête directe est bien compris, il est possible de passer à l'analyse des requêtes préparées, et en particulier, d'examiner pourquoi elles offrent une protection contre les injections SQL.

## Fonctionnement d'une requête préparée

### Exemple d'une requête préparée avec prepare()

La correction de l'injection SQL du code présenté en introduction peut se faire en utilisant les requêtes préparées de la manière suivante :&#x20;

```php
$city_name = $_GET['name'];
$sql = "SELECT name, population FROM cities WHERE name = ?";

$stmt = $cnx->prepare($sql);
$stmt->bind_param("s", $city_name);
$stmt->execute();
```

Cette fois, la tentative d'injection ne fonctionne pas, et aucun résultat n'est retourné par l'application.

```http
GET /city?name=' OR 1=1# HTTP/1.1
```

```http
GET /city?name=' UNION SELECT username,password FROM users# HTTP/1.1
```

Il est cependant intéressant de noter qu'une utilisation incorrecte des requêtes préparées peut également conduire à une injection SQL :&#x20;

```php
$city_name = $_GET['name'];
$sql = "SELECT name, population FROM cities WHERE name = '$city_name'";

$stmt = $cnx->prepare($sql);
$stmt->execute();
```

```http
GET /city?name=' OR 1=1# HTTP/1.1

City: Paris
Population: 2148327
City: Marseille
Population: 861635
City: Lyon
Population: 513275
City: Toulouse
Population: 479175
```

```http
GET /city?name=' UNION SELECT username,password FROM users# HTTP/1.1

City: azerty
Population: admin
```

Deux fonctions clés seront examinées dans la prochaine partie : la fonction `prepare()` ainsi que la fonction `bind_param()`.&#x20;

### Analyse du code PHP lors de l'exécution d'une requête préparée

#### Analyse de la fonction prepare()

Lors de l'appel à la méthode PHP `prepare()`, la fonction `mysqli_prepare()`, qui est un alias défini dans **ext/mysqli/mysqli.stub.php** présente dans le fichier **ext/mysqli/mysqli\_api.c**, est exécutée :&#x20;

```c
PHP_FUNCTION(mysqli_prepare)
{
    MY_MYSQL            *mysql;
    MY_STMT		*stmt;
    char		*query = NULL;
    size_t		query_len;
    zval		*mysql_link;
    MYSQLI_RESOURCE	*mysqli_resource;

    if (zend_parse_method_parameters(ZEND_NUM_ARGS(), getThis(), "Os",&mysql_link, mysqli_link_class_entry, &query, &query_len) == FAILURE) {
        RETURN_THROWS();
    }
    MYSQLI_FETCH_RESOURCE_CONN(mysql, mysql_link, MYSQLI_STATUS_VALID);

    stmt = (MY_STMT *)ecalloc(1,sizeof(MY_STMT));

    if ((stmt->stmt = mysql_stmt_init(mysql->mysql))) {
        if (mysql_stmt_prepare(stmt->stmt, query, query_len)) {
            /* mysql_stmt_close() clears errors, so we have to store them temporarily */
            MYSQLND_ERROR_INFO error_info = *mysql->mysql->data->error_info;
            mysql->mysql->data->error_info->error_list.head = NULL;
            mysql->mysql->data->error_info->error_list.tail = NULL;
            mysql->mysql->data->error_info->error_list.count = 0;
            mysqli_stmt_close(stmt->stmt, false);
            stmt->stmt = NULL;

            /* restore error messages */
            zend_llist_clean(&mysql->mysql->data->error_info->error_list);
            *mysql->mysql->data->error_info = error_info;
        }
    }

    /* don't initialize stmt->query with NULL, we ecalloc()-ed the memory */
    /* Get performance boost if reporting is switched off */
    if (stmt->stmt && query_len && (MyG(report_mode) & MYSQLI_REPORT_INDEX)) {
        stmt->query = estrdup(query);
    }

    /* don't join to the previous if because it won't work if mysql_stmt_prepare_fails */
    if (!stmt->stmt) {
        MYSQLI_REPORT_MYSQL_ERROR(mysql->mysql);
        efree(stmt);
        RETURN_FALSE;
    }

    mysqli_resource = (MYSQLI_RESOURCE *)ecalloc (1, sizeof(MYSQLI_RESOURCE));
    mysqli_resource->ptr = (void *)stmt;

    /* change status */
    mysqli_resource->status = MYSQLI_STATUS_VALID;
    MYSQLI_RETVAL_RESOURCE(mysqli_resource, mysqli_stmt_class_entry);
}
```

Après avoir passé les vérifications et la gestion des erreurs, les méthodes clés sont l'initialisation de la déclaration avec `mysql_stmt_init()` et sa préparation avec `mysql_stmt_prepare()`.

La fonction `mysql_stmt_init()` est issue de mysqlnd sous le nom `mysqlnd_stmt_init()` définie dans le fichier **ext/mysqlnd/mysqlnd\_libmysql\_compat.h** :

```c
#define mysql_stmt_init(r)    mysqlnd_stmt_init((r))
```

Puis dans le fichier **ext/mysqlnd/mysqlnd.h** :&#x20;

```c
#define mysqlnd_stmt_init(conn)    ((conn)->data)->m->stmt_init(((conn)->data))
```

Cette fonction, présente dans le fichier **ext/mysqlnd/mysqlnd\_connection.c**, crée simplement un objet `MYSQLND_STMT` , représentant une requête préparée associée à la connexion :&#x20;

<pre class="language-c"><code class="lang-c">MYSQLND_STMT *
<strong>MYSQLND_METHOD(mysqlnd_conn_data, stmt_init)(MYSQLND_CONN_DATA * const conn)
</strong>{
    MYSQLND_STMT * ret;
    DBG_ENTER("mysqlnd_conn_data::stmt_init");
    ret = conn->object_factory.get_prepared_statement(conn);
    DBG_RETURN(ret);
}
</code></pre>

De la même façon, la fonction `mysql_stmt_prepare()` est définie dans le fichier **ext/mysqlnd/mysqlnd\_libmysql\_compat.h** :&#x20;

```c
#define mysql_stmt_prepare(s,q,l)    mysqlnd_stmt_prepare((s), (q), (l))
```

Puis dans le fichier **ext/mysqlnd/mysqlnd.h** :&#x20;

```c
#define mysqlnd_stmt_prepare(stmt, q, qlen)    (stmt)->m->prepare((stmt), (q), (qlen))
```

Cette fonction est définie dans le fichier **ext/mysqlnd/mysqlnd\_ps.c** :&#x20;

```c
static enum_func_status
MYSQLND_METHOD(mysqlnd_stmt, prepare)(MYSQLND_STMT * const s, const char * const query, const size_t query_len)
{
    MYSQLND_STMT_DATA * stmt = s? s->data : NULL;
    MYSQLND_CONN_DATA * conn = stmt? stmt->conn : NULL;

    DBG_ENTER("mysqlnd_stmt::prepare");
    if (!stmt || !conn) {
        DBG_RETURN(FAIL);
    }
    DBG_INF_FMT("stmt=" ZEND_ULONG_FMT, stmt->stmt_id);
    DBG_INF_FMT("query=%s", query);

    UPSERT_STATUS_SET_AFFECTED_ROWS_TO_ERROR(stmt->upsert_status);
    UPSERT_STATUS_SET_AFFECTED_ROWS_TO_ERROR(conn->upsert_status);

    SET_EMPTY_ERROR(stmt->error_info);
    SET_EMPTY_ERROR(conn->error_info);

    if (stmt->state > MYSQLND_STMT_INITTED) {
        /*
            Create a new prepared statement and destroy the previous one.
        */
        MYSQLND_STMT * s_to_prepare = conn->m->stmt_init(conn);
        if (!s_to_prepare) {
            goto fail;
        }
        MYSQLND_STMT_DATA * stmt_to_prepare = s_to_prepare->data;

        /* swap */
        size_t real_size = sizeof(MYSQLND_STMT) + mysqlnd_plugin_count() * sizeof(void *);
        char * tmp_swap = mnd_emalloc(real_size);
        memcpy(tmp_swap, s, real_size);
        memcpy(s, s_to_prepare, real_size);
        memcpy(s_to_prepare, tmp_swap, real_size);
	mnd_efree(tmp_swap);
        {
            MYSQLND_STMT_DATA * tmp_swap_data = stmt_to_prepare;
            stmt_to_prepare = stmt;
            stmt = tmp_swap_data;
        }
        s_to_prepare->m->dtor(s_to_prepare, TRUE);
    }

    {
        enum_func_status ret = FAIL;
        const MYSQLND_CSTRING query_string = {query, query_len};

        ret = conn->command->stmt_prepare(conn, query_string);
        if (FAIL == ret) {
            COPY_CLIENT_ERROR(stmt->error_info, *conn->error_info);
            goto fail;
        }
    }

    if (FAIL == mysqlnd_stmt_read_prepare_response(s)) {
        goto fail;
    }

    if (stmt->param_count) {
        if (FAIL == mysqlnd_stmt_skip_metadata(s) || FAIL == mysqlnd_stmt_prepare_read_eof(s)) {
            goto fail;
        }
    }

    /*
        Read metadata only if there is actual result set.
        Beware that SHOW statements bypass the PS framework and thus they send
        no metadata at prepare.
    */
    if (stmt->field_count) {
        MYSQLND_RES * result = conn->m->result_init(stmt->field_count);
        if (!result) {
            SET_OOM_ERROR(conn->error_info);
            goto fail;
        }
        /* Allocate the result now as it is needed for the reading of metadata */
        stmt->result = result;

        result->conn = conn->m->get_reference(conn);

        result->type = MYSQLND_RES_PS_BUF;

        if (FAIL == result->m.read_result_metadata(result, conn) || FAIL == mysqlnd_stmt_prepare_read_eof(s)) {
            goto fail;
        }
    }

    stmt->state = MYSQLND_STMT_PREPARED;
    DBG_INF("PASS");
    DBG_RETURN(PASS);

    fail:
	DBG_INF("FAIL");
	DBG_RETURN(FAIL);
}
```

La première section intéressante est l'envoi de la commande au serveur MySQL via un appel à la fonction `stmt_prepare()`, présente dans le fichier **ext/mysqlnd/mysqlnd\_commands.c**, de l'objet `command` :&#x20;

```c
static enum_func_status
MYSQLND_METHOD(mysqlnd_command, stmt_prepare)(MYSQLND_CONN_DATA * const conn, const MYSQLND_CSTRING query)
{
    func_mysqlnd_protocol_payload_decoder_factory__send_command send_command = conn->payload_decoder_factory->m.send_command;
    enum_func_status ret = FAIL;

    DBG_ENTER("mysqlnd_command::stmt_prepare");

    ret = send_command(conn->payload_decoder_factory, COM_STMT_PREPARE, (const zend_uchar*) query.s, query.l, FALSE,
        &conn->state,
        conn->error_info,
        conn->upsert_status,
        conn->stats,
        conn->m->send_close,
        conn);

    DBG_RETURN(ret);
}
```

Cet appel entraîne à son tour l'exécution de la fonction `send_command()`, précédemment analysée lors de l'étude des requêtes directes, et qui transmet la requête SQL au serveur MySQL en utilisant `PACKET_WRITE()`:

{% hint style="info" %}
A noter toutefois que l'argument `command` possède cette fois la valeur `COM_STMT_PREPARE`.
{% endhint %}

```c
static enum_func_status
MYSQLND_METHOD(mysqlnd_protocol, send_command)(
    MYSQLND_PROTOCOL_PAYLOAD_DECODER_FACTORY * payload_decoder_factory,
    const enum php_mysqlnd_server_command command,
    const zend_uchar * const arg, const size_t arg_len,
    const bool silent,

    struct st_mysqlnd_connection_state * connection_state,
    MYSQLND_ERROR_INFO * error_info,
    MYSQLND_UPSERT_STATUS * upsert_status,
    MYSQLND_STATS * stats,
    func_mysqlnd_conn_data__send_close send_close,
    void * send_close_ctx)
{
    enum_func_status ret = PASS;
    MYSQLND_PACKET_COMMAND cmd_packet;
    enum mysqlnd_connection_state state;
    DBG_ENTER("mysqlnd_protocol::send_command");
    DBG_INF_FMT("command=%s silent=%u", mysqlnd_command_to_text[command], silent);
    DBG_INF_FMT("server_status=%u", UPSERT_STATUS_GET_SERVER_STATUS(upsert_status));
    DBG_INF_FMT("sending %zu bytes", arg_len + 1); /* + 1 is for the command */
    state = connection_state->m->get(connection_state);

    switch (state) {
        case CONN_READY:
            break;
        case CONN_QUIT_SENT:
            SET_CLIENT_ERROR(error_info, CR_SERVER_GONE_ERROR, UNKNOWN_SQLSTATE, mysqlnd_server_gone);
            DBG_ERR("Server is gone");
            DBG_RETURN(FAIL);
        default:
            SET_CLIENT_ERROR(error_info, CR_COMMANDS_OUT_OF_SYNC, UNKNOWN_SQLSTATE, mysqlnd_out_of_sync);
            DBG_ERR_FMT("Command out of sync. State=%u", state);
            DBG_RETURN(FAIL);
    }

    UPSERT_STATUS_SET_AFFECTED_ROWS_TO_ERROR(upsert_status);
    SET_EMPTY_ERROR(error_info);

    payload_decoder_factory->m.init_command_packet(&cmd_packet);

    cmd_packet.command = command;
    if (arg && arg_len) {
        cmd_packet.argument.s = (char *) arg;
        cmd_packet.argument.l = arg_len;
    }

    MYSQLND_INC_CONN_STATISTIC(stats, STAT_COM_QUIT + command - 1 /* because of COM_SLEEP */ );

    if (! PACKET_WRITE(payload_decoder_factory->conn, &cmd_packet)) {
        if (!silent && error_info->error_no != CR_SERVER_GONE_ERROR) {
            DBG_ERR_FMT("Error while sending %s packet", mysqlnd_command_to_text[command]);
            php_error(E_WARNING, "Error while sending %s packet. PID=%d", mysqlnd_command_to_text[command], getpid());
        }
        connection_state->m->set(connection_state, CONN_QUIT_SENT);
        send_close(send_close_ctx);
        DBG_ERR("Server is gone");
        ret = FAIL;
    }
    PACKET_FREE(&cmd_packet);
    DBG_RETURN(ret);
}
```

En supposant une mauvaise utilisation des requêtes préparées, comme illustré au début de cette section, MySQL ne peut fournir aucune protection et l'application sera donc bel et bien vulnérable lors de l'appel à la méthode `execute()` :&#x20;

```php
$city_name = $_GET['name'];
$sql = "SELECT name, population FROM cities WHERE name = '$city_name'";

$stmt = $cnx->prepare($sql);
$stmt->execute();
```

```http
GET /city?name=' OR 1=1# HTTP/1.1

City: Paris
Population: 2148327
City: Marseille
Population: 861635
City: Lyon
Population: 513275
City: Toulouse
Population: 479175
```

```sql
SELECT id, name, population FROM cities WHERE name = '' OR 1=1#'
```

La deuxième section importante concerne le traitement des paramètres de marque (parameter markers), représentés par le caractère "?", qui est géré par la méthode `mysqlnd_stmt_read_prepare_response()` présente dans le fichier **ext/mysqlnd/mysqlnd\_ps.c** :&#x20;

```c
static enum_func_status
mysqlnd_stmt_read_prepare_response(MYSQLND_STMT * s)
{
    MYSQLND_STMT_DATA * stmt = s? s->data : NULL;
    MYSQLND_CONN_DATA * conn = stmt? stmt->conn : NULL;
    MYSQLND_PACKET_PREPARE_RESPONSE prepare_resp;
    enum_func_status ret = FAIL;

    DBG_ENTER("mysqlnd_stmt_read_prepare_response");
    if (!stmt || !conn) {
        DBG_RETURN(FAIL);
    }
    DBG_INF_FMT("stmt=" ZEND_ULONG_FMT, stmt->stmt_id);

    conn->payload_decoder_factory->m.init_prepare_response_packet(&prepare_resp);

    if (FAIL == PACKET_READ(conn, &prepare_resp)) {
        goto done;
    }

    if (0xFF == prepare_resp.error_code) {
        COPY_CLIENT_ERROR(stmt->error_info, prepare_resp.error_info);
	COPY_CLIENT_ERROR(conn->error_info, prepare_resp.error_info);
        goto done;
    }
    ret = PASS;
    stmt->stmt_id = prepare_resp.stmt_id;
    UPSERT_STATUS_SET_WARNINGS(conn->upsert_status, prepare_resp.warning_count);
    UPSERT_STATUS_SET_AFFECTED_ROWS(stmt->upsert_status, 0);  /* be like libmysql */
    stmt->field_count = conn->field_count = prepare_resp.field_count;
    stmt->param_count = prepare_resp.param_count;
done:
    PACKET_FREE(&prepare_resp);

    DBG_RETURN(ret);
}
```

Cette méthode lit et traite la réponse du serveur MySQL après l'envoi de la commande de préparation, et récupère le nombre de paramètres de la requête, qui est ensuite stocké dans `param_count` de l'objet `stmt`. Une fois la requête préparée et le nombre de paramètres déterminé, il est possible de se concentrer sur l'analyse de la méthode `bind_param()`.

#### Analyse de la fonction bind\_param()

La méthode permettant de lier les paramètres est la fonction `mysqli_stmt_bind_param()` (via un alias défini dans **ext/mysqli/mysqli.stub.php**) présente dans le fichier **ext/mysqli/mysqli\_api.c** dont le code est le suivant :&#x20;

```c
PHP_FUNCTION(mysqli_stmt_bind_param)
{
    zval        *args;
    uint32_t	argc;
    MY_STMT	*stmt;
    zval	*mysql_stmt;
    char	*types;
    size_t	types_len;

    if (zend_parse_method_parameters(ZEND_NUM_ARGS(), getThis(), "Os*", &mysql_stmt, mysqli_stmt_class_entry, &types, &types_len, &args, &argc) == FAILURE) {
        RETURN_THROWS();
    }

    MYSQLI_FETCH_RESOURCE_STMT(stmt, mysql_stmt, MYSQLI_STATUS_VALID);

    if (!types_len) {
        zend_argument_must_not_be_empty_error(ERROR_ARG_POS(2));
        RETURN_THROWS();
    }

    if (types_len != (size_t) argc) {
        /* number of bind variables doesn't match number of elements in type definition string */
        zend_argument_count_error("The number of elements in the type definition string must match the number of bind variables");
        RETURN_THROWS();
    }

    if (types_len != mysql_stmt_param_count(stmt->stmt)) {
        zend_argument_count_error("The number of variables must match the number of parameters in the prepared statement");
        RETURN_THROWS();
    }

    RETVAL_BOOL(mysqli_stmt_bind_param_do_bind(stmt, argc, args, types, ERROR_ARG_POS(2)) == PASS);
    MYSQLI_REPORT_STMT_ERROR(stmt->stmt);
}
```

L'appel à la méthode `MYSQLI_FETCH_RESOURCE_STMT()` permet de récupérer l'élément `stmt` associé, c'est-à-dire la requête préparée. Puis deux vérifications sont effectuées :&#x20;

* Le nombre de paramètres fournis à `bind_param()` doit correspondre à ceux définis par la chaîne de caractères (ex. à "ssi" doit correspondre à trois paramètres).
* Le nombre de paramètres fournis à `bind_param()` doit correspondre à ceux déclarés à la préparation de la requête (les caractères "?").

Une fois ces vérifications effectuées, la liaison est effectuée grâce à la méthode `mysqli_stmt_bind_param_do_bind()` présente dans le même fichier :&#x20;

```c
static enum_func_status mysqli_stmt_bind_param_do_bind(MY_STMT *stmt, uint32_t num_vars, zval *args, const char * const types, unsigned int arg_num)
{
    MYSQLND_PARAM_BIND	*params;
    enum_func_status	ret = FAIL;

    /* If no params -> skip binding and return directly */
    if (num_vars == 0) {
        return PASS;
    }
    params = mysqlnd_stmt_alloc_param_bind(stmt->stmt);
    if (!params) {
        goto end;
    }
    for (uint32_t i = 0; i < num_vars; i++) {
        uint8_t type;
        switch (types[i]) {
            case 'd': /* Double */
                type = MYSQL_TYPE_DOUBLE;
		break;
            case 'i': /* Integer */
#if SIZEOF_ZEND_LONG==8
                type = MYSQL_TYPE_LONGLONG;
#elif SIZEOF_ZEND_LONG==4
                type = MYSQL_TYPE_LONG;
#endif
                break;
                case 'b': /* Blob (send data) */
                    type = MYSQL_TYPE_LONG_BLOB;
                    break;
                case 's': /* string */
                    type = MYSQL_TYPE_VAR_STRING;
                    break;
                default:
                    zend_argument_value_error(arg_num, "must only contain the \"b\", \"d\", \"i\", \"s\" type specifiers");
                    ret = FAIL;
                    mysqlnd_stmt_free_param_bind(stmt->stmt, params);
                goto end;
        }
        ZVAL_COPY_VALUE(&params[i].zv, &args[i]);
        params[i].type = type;
    }
    ret = mysqlnd_stmt_bind_param(stmt->stmt, params);

end:
    return ret;
}
```

L'appel à la fonction `mysqlnd_stmt_alloc_param_bind()` permet d'allouer de la mémoire afin de stocker les paramètres à lier. Elle est définie dans le fichier **ext/mysqlnd/mysqlnd.h** :&#x20;

```c
#define mysqlnd_stmt_alloc_param_bind(stmt)        (stmt)->m->alloc_parameter_bind((stmt))
```

Puis dans **ext/mysqlnd/mysqlnd\_structs.h** :&#x20;

```c
MYSQLND_CLASS_METHODS_TYPE(mysqlnd_stmt)
{
    ...
    func_mysqlnd_stmt__alloc_param_bind alloc_parameter_bind;
    ...
}
```

Son code est le suivant (`mnd_ecalloc()` permet d'allouer dynamiquement la mémoire nécessaire) :

```c
static MYSQLND_PARAM_BIND *
MYSQLND_METHOD(mysqlnd_stmt, alloc_param_bind)(MYSQLND_STMT * const s)
{
    MYSQLND_STMT_DATA * stmt = s? s->data : NULL;
    DBG_ENTER("mysqlnd_stmt::alloc_param_bind");
    if (!stmt) {
        DBG_RETURN(NULL);
    }
    DBG_RETURN(mnd_ecalloc(stmt->param_count, sizeof(MYSQLND_PARAM_BIND)));
}
```

Vient ensuite une vérification du type du paramètre à lier ("s" pour string, "i" pour integer, "d" pour double et "b" pour blob). Puis un appel à la méthode `mysqlnd_stmt_bind_param()` pour lier les paramètres préparés à la déclaration de la requête, qui, comme l'indique le fichier **ext/mysqlnd/mysqlnd.h** est en réalité un appel à `bind_parameters()` :

```cpp
#define mysqlnd_stmt_bind_param(stmt,bind)        (stmt)->m->bind_parameters((stmt), (bind))
```

Le code de cette méthode est présent dans le fichier **ext/mysqldn/mysqlnd\_ps.c** :&#x20;

```c
static enum_func_status
MYSQLND_METHOD(mysqlnd_stmt, bind_parameters)(MYSQLND_STMT * const s, MYSQLND_PARAM_BIND * const param_bind)
{
    MYSQLND_STMT_DATA * stmt = s? s->data : NULL;
    MYSQLND_CONN_DATA * conn = stmt? stmt->conn : NULL;

    DBG_ENTER("mysqlnd_stmt::bind_param");
    if (!stmt || !conn) {
        DBG_RETURN(FAIL);
    }
    DBG_INF_FMT("stmt=" ZEND_ULONG_FMT " param_count=%u", stmt->stmt_id, stmt->param_count);

    if (stmt->state < MYSQLND_STMT_PREPARED) {
        SET_CLIENT_ERROR(stmt->error_info, CR_NO_PREPARE_STMT, UNKNOWN_SQLSTATE, mysqlnd_stmt_not_prepared);
        DBG_ERR("not prepared");
        if (param_bind) {
            s->m->free_parameter_bind(s, param_bind);
        }
        DBG_RETURN(FAIL);
    }

    SET_EMPTY_ERROR(stmt->error_info);
    SET_EMPTY_ERROR(conn->error_info);

    if (stmt->param_count) {
        unsigned int i = 0;

        if (!param_bind) {
            SET_CLIENT_ERROR(stmt->error_info, CR_COMMANDS_OUT_OF_SYNC, UNKNOWN_SQLSTATE, "Re-binding (still) not supported");
            DBG_ERR("Re-binding (still) not supported");
            DBG_RETURN(FAIL);
        } else if (stmt->param_bind) {
            DBG_INF("Binding");
            /*
              There is already result bound.
              Forbid for now re-binding!!
            */
            for (i = 0; i < stmt->param_count; i++) {
                /*
                  We may have the last reference, then call zval_ptr_dtor() or we may leak memory.
                  Switching from bind_one_parameter to bind_parameters may result in zv being NULL
                */
                zval_ptr_dtor(&stmt->param_bind[i].zv);
            }
            if (stmt->param_bind != param_bind) {
                s->m->free_parameter_bind(s, stmt->param_bind);
            }
        }

        stmt->param_bind = param_bind;
        for (i = 0; i < stmt->param_count; i++) {
            /* The client will use stmt_send_long_data */
            DBG_INF_FMT("%u is of type %u", i, stmt->param_bind[i].type);
            /* Prevent from freeing */
            /* Don't update is_ref, or we will leak during conversion */
            Z_TRY_ADDREF(stmt->param_bind[i].zv);
            stmt->param_bind[i].flags = 0;
            if (stmt->param_bind[i].type == MYSQL_TYPE_LONG_BLOB) {
                stmt->param_bind[i].flags &= ~MYSQLND_PARAM_BIND_BLOB_USED;
            }
        }
        stmt->send_types_to_server = 1;
    } else if (param_bind && param_bind != stmt->param_bind) {
        s->m->free_parameter_bind(s, param_bind);
    }
    DBG_INF("PASS");
    DBG_RETURN(PASS);
}
```

La fonction débute par une série de vérifications, mais c'est la ligne suivante qui établit le lien entre les paramètres fournis par le développeur à la méthode `bind_param()` et la déclaration de la requête SQL (`stmt`) :

```
stmt->param_bind = param_bind;
```

Les paramètres sont maintenant liés à la requête. La fonction `execute()` prépare ces paramètres et les place dans un buffer dédié avant de les envoyer à MySQL. Lors de l'exécution de la requête, le serveur MySQL est capable de distinguer clairement entre la structure de la requête SQL et les valeurs des paramètres, qui sont traitées comme des données et non comme du code. Cette séparation garantit que même si les valeurs contiennent des caractères spéciaux, elles ne seront pas interprétées comme du code SQL, empêchant ainsi toute injection malveillante.

## Conclusion

Les requêtes préparées constituent probablement l'une des méthodes les plus efficaces pour se protéger contre les injections SQL (les procédures stockées peuvent également être utilisées à cette fin). Cette protection résulte d'une séparation claire et sécurisée entre le code (la requête SQL) et les données, potentiellement non fiables, empêchant ainsi toute interférence entre les deux.
