# React Data Table con Material UI y Spark of Joy

Cuando desea visualizar grandes cantidades de datos uniformes, los gráficos no funcionan bien porque ocultan eficazmente información sobre elementos de datos individuales. **Sin embargo, es el caso cuando las tablas de datos son útiles.**

En este tutorial, aprenderemos cómo mostrar cargas masivas de datos en una tabla de datos construida desde cero en React. Exploraremos cómo obtener datos de una base de datos a través de una API y visualizarlos en una tabla de datos con características esenciales como filtrado, clasificación, etc.

Usaremos [Material UI](https://material-ui.com/) porque es el framework de interfaz de usuario más popular para React. Creado con la inspiración del [Material Design](https://material.io/design/) de Google, proporciona una gran cantidad de componentes que podemos usar para obtener una interfaz de usuario atractiva.

## Cómo construir una tabla de datos

¡Aquí está nuestro plan para hoy!

- [Prepare los datos](#Prepare-los-datos-en-la-base-de-datos) en la base de datos: ¡muchos datos!
- [Inicie una API](#Lanzar-una-API-para-trabajar-con-datos) para trabajar con esos datos, de forma rápida y sencilla
- [Cree una aplicación](#Crea-una-aplicacion-con-React-⚛️) con React y Material UI
- [Construye una tabla de datos básica](#Construye-una-tabla-de-datos-basica)
- [Amplíe la tabla de datos](#Formato-de-celda-personalizado) con varias funciones, paso a paso

¿Suena bien? ¡Vamos!

Antes de sumergirse en la piratería, vea una captura de pantalla de la tabla de datos que vamos a crear. Además, consulte el [código fuente completo](https://github.com/cube-js/cube.js/tree/master/examples/react-data-table) disponible en GitHub.

![](/images/1.png)

## Prepare los datos en la base de datos

Supongo que usaremos uno de los almacenes de datos SQL más populares: la base de datos [PostgreSQL](https://www.postgresql.org/) . Asegúrese de tener instalado PostgreSQL. (De lo contrario, puede que algún día deje de ser tan popular).

Ahora podemos descargar e importar un conjunto de datos de comercio electrónico de muestra cuidadosamente preparado para PostgreSQL. El conjunto de datos está relacionado con una empresa de comercio electrónico imaginaria que quiere rastrear sus pedidos y sus estados:

```bash
$ curl <http://cube.dev/downloads/ecom-dump.sql> > ecom-dump.sql
$ createdb ecom
$ psql --dbname ecom -f ecom-dump.sql
```

Entonces, ¡la base de datos está lista! Procedamos a...

## Lanzar una API para trabajar con datos

Usaremos [Cube.js](https://cube.dev/?utm_source=dev-to&utm_medium=post&utm_campaign=react-data-table) para nuestra API. Cube.js es una plataforma de API analítica de código abierto que ayuda a crear APIs para almacenes de datos SQL y crear aplicaciones analíticas. Elimina todo el ajetreo de construir la capa API, generar SQL y consultar la base de datos. También proporciona muchas características de nivel de producción, como almacenamiento en caché de varios niveles para un rendimiento óptimo, tenencia múltiple, seguridad y más.

Entonces, lancemos una API sobre nuestra base de datos con Cube.js en unos simples pasos.

**Primero, necesitamos instalar la utilidad de línea de comandos (CLI)** de Cube.js. Para mayor comodidad, instalémoslo globalmente en nuestra máquina.

```terminal
$ npm install -g cubejs-cli
```

Luego, con la CLI instalada, podemos crear un backend básico ejecutando un solo comando. Cube.js es compatible con [todas las bases de datos populares](https://cube.dev/docs/getting-started?utm_source=dev-to&utm_medium=post&utm_campaign=react-data-table#2-connect-to-your-database), por lo que podemos preconfigurar el backend para que funcione con PostgreSQL:

```terminal
$ cubejs create <project name> -d <database type>
```

Para crear el backend, ejecutamos este comando:

```terminal
$ cubejs create react-data-table -d postgres
```

Ahora necesitamos [conectarlo](https://cube.dev/docs/connecting-to-the-database?utm_source=dev-to&utm_medium=post&utm_campaign=react-data-table#configuring-connection-for-cube-js-cli-created-apps) a la base de datos. Para hacer eso, proporcionamos algunas opciones a través del archivo `.env` en la raíz de la carpeta del proyecto Cube.js (`react-data-table`):

```env
CUBEJS_DB_NAME=ecom
CUBEJS_DB_TYPE=postgres
CUBEJS_API_SECRET=secre
```

![](/images/start.gif)

¡Ahora podemos ejecutar el backend!

**En el modo de desarrollo, el backend también ejecutará Cube.js Playground**. Eso es realmente genial. Cube.js Playground, una aplicación web que ahorra tiempo y que ayuda a crear un esquema de datos, probar las consultas y generar un modelo de panel de React. Ejecute el siguiente comando en la carpeta del proyecto Cube.js:

```terminal
$ node index.js
```

A continuación, abra [http://localhost:4000](http://localhost:4000) en su navegador.

![](/images/demo.gif)

**Usaremos Cube.js Playground para crear un esquema de datos.** Es esencialmente un código JavaScript que describe los datos de manera declarativa, define entidades analíticas como medidas y dimensiones, y las asigna a consultas SQL. A continuación, se muestra un ejemplo del esquema que se puede utilizar para describir los datos de los productos.

```sql
cube(`Products`, {
  sql: `SELECT * FROM public.products`,

  measures: {
    count: {
      type: `count`
    }
  },

  dimensions: {
    name: {
      sql: `name`,
      type: `string`
    },

    id: {
      sql: `id`,
      type: `number`,
      primaryKey: true,
      shown: true
    },

    description: {
      sql: `description`,
      type: `string`
    },

    createdAt: {
      sql: `created_at`,
      type: `time`
    }
  }
});
```

Cube.js puede generar un esquema de datos simple basado en las tablas de la base de datos. Si ya tiene un conjunto de tablas no trivial en su base de datos, considere usar la generación de esquemas de datos porque puede ahorrar mucho tiempo.

Por lo tanto, vamos a navegar a la pestaña Schema del Cube.js Playground, seleccione el grupo `public` en la vista de árbol, seleccione las tablas `line_items`, `orders`, `products`, y `users`, y haga clic en “public” Como resultado, tendremos 4 archivos generados en la carpeta `schema`, exactamente un archivo de esquema por tabla.

![](/images/2.png)

**Una vez que se genera el esquema, podemos consultar los datos a través de Cube.js Playground.** Para hacerlo, navegue a la pestaña "Build" y seleccione algunas medidas y dimensiones del esquema. Parece magia, ¿no?

La pestaña "Create" es un lugar donde puede crear gráficos de muestra utilizando diferentes bibliotecas de visualización e inspeccionar cada aspecto de cómo se creó ese gráfico, desde el SQL generado hasta el código JavaScript para representar el gráfico. También puede inspeccionar la consulta de Cube.js codificada con JSON que se envía al backend de Cube.js.

![](/images/3.png)

Está bien, estamos listos. La API está lista, ahora vamos a...

## Crea una aplicacion con React ⚛️

¡Grandes noticias!

**Cube.js Playground puede generar una plantilla para cualquier framework de interfaz y biblioteca de gráficos elegidos.** Para crear una plantilla para nuestra aplicación, navegue hasta la "Dashboard App" y use estas opciones:

- Framework: `React`
- Main Template:` React Material UI Static`
- Charting Library: `Chart.js`

![](/images/1.gif)

¡Felicidades! Ahora tenemos la carpeta `dashboard-app` en nuestro proyecto. Esta carpeta contiene todo el código de la interfaz que vamos a ampliar.

Antes de continuar, hagamos el cambio más crucial: muestre en el título que estamos creando una tabla de datos.

Para hacerlo, cambie algunas líneas en el archivo `public/index.html` del `dashboard-app` de la siguiente manera:

```html
// ... - <title>React App</title> + <title>React Data Table</title> +
<style>
  +      body {
  +          background-color: #eeeeee;
  +          margin: 0;
  +      }
  +
</style>
// ...
```

Además, instalemos algunas dependencias de `dashboard-app` que facilitarán nuestra tarea de construir una tabla de datos:

```node
$ npm install --save react-perfect-scrollbar @material-ui/pickers
```

Entonces, ahora estamos listos para...

## Construye una tabla de datos basica

Es genial tener muchos datos en la tabla, ¿verdad? Entonces, vayamos a buscarlos a través de la API.

Para hacerlo, vamos a definir una serie de nuevas métricas: cantidad de artículos en un pedido (su tamaño), el precio de un pedido y el nombre completo de un usuario. Con Cube.js, es muy fácil:

Primero, agreguemos el nombre completo en el esquema "Users" en el archivo `schema/Users.js`. Para crear el nombre completo, concatenamos el nombre y el apellido usando la función SQL `CONCAT`:

```sql
cube(`Users`, {
  sql: `SELECT * FROM public.users`,

// ...

  dimensions: {

// ...

    id: {
+      shown: true,
      sql: `id`,
      type: `number`,
      primaryKey: true
    },

    firstName: {
      sql: `first_name`,
      type: `string`
    },

    lastName: {
      sql: `last_name`,
      type: `string`
    },

+    fullName: {
+      sql: `CONCAT(${firstName}, ' ', ${lastName})`,
+      type: `string`
+    },

// ...
```

Luego, agreguemos otras medidas al esquema "Pedidos" en el archivo `schema/Orders.js`.

Para estas medidas, usaremos la función de [subconsulta](https://cube.dev/docs/subquery?utm_source=dev-to&utm_medium=post&utm_campaign=react-data-table#top) de Cube.js. Puede utilizar dimensiones de subconsultas para hacer referencia a medidas de otros cubos dentro de una dimensión. A continuación, se explica cómo definir dichas dimensiones:

```sql
cube(`Orders`, {
  sql: `SELECT * FROM public.orders`,

  dimensions: {

// ...

    id: {
+      shown: true,
      sql: `id`,
      type: `number`,
      primaryKey: true
    },

    createdAt: {
      sql: `created_at`,
      type: `time`
    },

+    size: {
+      sql: `${LineItems.count}`,
+      subQuery: true,
+      type: 'number'
+    },
+
+    price: {
+      sql: `${LineItems.price}`,
+      subQuery: true,
+      type: 'number'
+    },

    completedAt: {
      sql: `completed_at`,
      type: `time`
    }
  }
});
```

¡Casi estámos allí! Entonces, para mostrar la tabla de datos, reemplacemos el archivo `src/pages/DashboardPage.js` con el siguiente contenido:

```javascript
import React from "react";
import { makeStyles } from "@material-ui/styles";

import Table from "../components/Table.js";

const useStyles = makeStyles((theme) => ({
  root: { padding: 15 },
  content: { marginTop: 15 },
}));

const Dashboard = () => {
  const classes = useStyles();

  const query = {
    timeDimensions: [
      {
        dimension: "Orders.createdAt",
        granularity: "day",
      },
    ],
    dimensions: [
      "Users.id",
      "Orders.id",
      "Orders.size",
      "Users.fullName",
      "Users.city",
      "Orders.price",
      "Orders.status",
      "Orders.createdAt",
    ],
  };

  return (
    <div className={classes.root}>
      <div className={classes.content}>
        <Table query={query} />
      </div>
    </div>
  );
};

export default Dashboard;
```

Tenga en cuenta que ahora este archivo contiene una consulta Cube.js que se explica por sí misma: solo le estamos pidiendo a la API que devuelva una serie de dimensiones, y lo hace.

Los cambios en el `Dashboard` son mínimos, sin embargo, toda la magia de renderizar una tabla de datos ocurre dentro del componente `<Table />`, y los cambios en el resultado de la consulta se reflejan en la tabla.

Creemos este componente `<Table />` en el archivo `src/components/Table.js` con el siguiente contenido:

```javascript
import React, { useState } from "react";
import clsx from "clsx";
import PropTypes from "prop-types";
import moment from "moment";
import PerfectScrollbar from "react-perfect-scrollbar";
import { makeStyles } from "@material-ui/styles";
import Typography from "@material-ui/core/Typography";
import { useCubeQuery } from "@cubejs-client/react";
import CircularProgress from "@material-ui/core/CircularProgress";
import {
  Card,
  CardActions,
  CardContent,
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableRow,
  TablePagination,
} from "@material-ui/core";

const useStyles = makeStyles((theme) => ({
  root: {
    padding: 0,
  },
  content: {
    padding: 0,
  },
  inner: {
    minWidth: 1050,
  },
  nameContainer: {
    display: "flex",
    alignItems: "baseline",
  },
  status: {
    marginRight: 15,
  },
  actions: {
    justifyContent: "flex-end",
  },
}));

const TableComponent = (props) => {
  const { className, query, cubejsApi, ...rest } = props;

  const classes = useStyles();

  const [rowsPerPage, setRowsPerPage] = useState(10);
  const [page, setPage] = useState(0);

  const tableHeaders = [
    { text: "Full Name", value: "Users.fullName" },
    { text: "User city", value: "Users.city" },
    { text: "Order price", value: "Orders.price" },
    { text: "Status", value: "Orders.status" },
    { text: "Created at", value: "Orders.createdAt" },
  ];
  const { resultSet, error, isLoading } = useCubeQuery(query, { cubejsApi });
  if (isLoading) {
    return (
      <div
        style={{
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
        }}
      >
        <CircularProgress color="secondary" />
      </div>
    );
  }
  if (error) {
    return <pre>{error.toString()}</pre>;
  }
  if (resultSet) {
    let orders = resultSet.tablePivot();

    const handlePageChange = (event, page) => {
      setPage(page);
    };
    const handleRowsPerPageChange = (event) => {
      setRowsPerPage(event.target.value);
    };

    return (
      <Card {...rest} padding={"0"} className={clsx(classes.root, className)}>
        <CardContent className={classes.content}>
          <PerfectScrollbar>
            <div className={classes.inner}>
              <Table>
                <TableHead className={classes.head}>
                  <TableRow>
                    {tableHeaders.map((item) => (
                      <TableCell
                        key={item.value + Math.random()}
                        className={classes.hoverable}
                      >
                        <span>{item.text}</span>
                      </TableCell>
                    ))}
                  </TableRow>
                </TableHead>
                <TableBody>
                  {orders
                    .slice(page * rowsPerPage, page * rowsPerPage + rowsPerPage)
                    .map((obj) => (
                      <TableRow
                        className={classes.tableRow}
                        hover
                        key={obj["Orders.id"]}
                      >
                        <TableCell>{obj["Orders.id"]}</TableCell>
                        <TableCell>{obj["Orders.size"]}</TableCell>
                        <TableCell>{obj["Users.fullName"]}</TableCell>
                        <TableCell>{obj["Users.city"]}</TableCell>
                        <TableCell>{"$ " + obj["Orders.price"]}</TableCell>
                        <TableCell>{obj["Orders.status"]}</TableCell>
                        <TableCell>
                          {moment(obj["Orders.createdAt"]).format("DD/MM/YYYY")}
                        </TableCell>
                      </TableRow>
                    ))}
                </TableBody>
              </Table>
            </div>
          </PerfectScrollbar>
        </CardContent>
        <CardActions className={classes.actions}>
          <TablePagination
            component="div"
            count={orders.length}
            onChangePage={handlePageChange}
            onChangeRowsPerPage={handleRowsPerPageChange}
            page={page}
            rowsPerPage={rowsPerPage}
            rowsPerPageOptions={[5, 10, 25, 50, 100]}
          />
        </CardActions>
      </Card>
    );
  } else {
    return null;
  }
};

TableComponent.propTypes = {
  className: PropTypes.string,
  query: PropTypes.object.isRequired,
};

export default TableComponent;
```

¡Finalmente! Aquí está la tabla de datos que estábamos esperando:

![](/images/4.png)

Se ve genial, ¿no?

¡Tenga en cuenta que en realidad no es tan básico! Tiene una paginación incorporada que permite mostrar y navegar grandes cantidades de datos.

Sin embargo, parece grisáceo y sombrío. Entonces, agreguemos color y extendamos la tabla con...

## Formato de celda personalizado

La tabla contiene los estados de los pedidos que se muestran como texto en este momento. ¡Reemplácelos con un componente personalizado!

La idea es visualizar el estado de un pedido con un punto de colores. Para eso, crearemos un componente personalizado `<StatusBullet />`. Creemos este componente en el archivo `src/components/StatusBullet.js` con el siguiente contenido:

```javascript
import React from "react";
import PropTypes from "prop-types";
import clsx from "clsx";
import { makeStyles } from "@material-ui/styles";

const useStyles = makeStyles((theme) => ({
  root: {
    display: "inline-block",
    borderRadius: "50%",
    flexGrow: 0,
    flexShrink: 0,
  },
  sm: {
    height: 15,
    width: 15,
  },
  md: {
    height: 15,
    width: 15,
  },
  lg: {
    height: 15,
    width: 15,
  },
  neutral: { backgroundColor: "#fff" },
  primary: { backgroundColor: "#ccc" },
  info: { backgroundColor: "#3cc" },
  warning: { backgroundColor: "#cc3" },
  danger: { backgroundColor: "#c33" },
  success: { backgroundColor: "#3c3" },
}));

const StatusBullet = (props) => {
  const { className, size, color, ...rest } = props;

  const classes = useStyles();

  return (
    <span
      {...rest}
      className={clsx(
        {
          [classes.root]: true,
          [classes[size]]: size,
          [classes[color]]: color,
        },
        className
      )}
    />
  );
};

StatusBullet.propTypes = {
  className: PropTypes.string,
  color: PropTypes.oneOf([
    "neutral",
    "primary",
    "info",
    "success",
    "warning",
    "danger",
  ]),
  size: PropTypes.oneOf(["sm", "md", "lg"]),
};

StatusBullet.defaultProps = {
  size: "md",
  color: "default",
};

export default StatusBullet;
```

Para que funcione, necesitaremos aplicar algunos cambios mínimos a la tabla de datos. Modifiquemos el `src/components/Table.js` de la siguiente manera:

```javascript
// ...

} from "@material-ui/core";

import StatusBullet from "./StatusBullet";

const statusColors = {
  completed: "success",
  processing: "info",
  shipped: "danger"
};

const useStyles = makeStyles(theme => ({

// ...

<TableCell>
+  <StatusBullet
+    className={classes.status}
+    color={statusColors[obj["Orders.status"]]}
+    size="sm"
+  />
  {obj["Orders.status"]}
</TableCell>

// ...
```

¡Lindo! Ahora tenemos una tabla que muestra información sobre todos los pedidos y tiene un toque colorido:

![](/images/5.png)

## Filtrar los datos

Sin embargo, es difícil explorar estas órdenes utilizando solo los controles proporcionados. Para solucionar este problema, agregaremos una barra de herramientas completa con filtros y haremos que nuestra tabla sea interactiva.

Primero, agreguemos algunas dependencias. Ejecute el comando en la carpeta `dashboard-app`:

```node
npm install --save @date-io/date-fns@1.x date-fns @date-io/moment@1.x moment
```

Luego, cree el componente `<Toolbar />` en el archivo `src/components/Toolbar.js` con el siguiente contenido:

```javascript
import "date-fns";
import React from "react";
import PropTypes from "prop-types";
import { makeStyles } from "@material-ui/styles";
import Grid from "@material-ui/core/Grid";
import Typography from "@material-ui/core/Typography";
import Tab from "@material-ui/core/Tab";
import Tabs from "@material-ui/core/Tabs";
import withStyles from "@material-ui/core/styles/withStyles";

const AntTabs = withStyles({
  indicator: {},
})(Tabs);
const AntTab = withStyles((theme) => ({
  root: {
    textTransform: "none",
    minWidth: 25,
    fontSize: 12,
    fontWeight: theme.typography.fontWeightRegular,
    marginRight: 0,
    opacity: 0.6,
    "&:hover": {
      opacity: 1,
    },
    "&$selected": {
      fontWeight: theme.typography.fontWeightMedium,
      outline: "none",
    },
    "&:focus": {
      outline: "none",
    },
  },
  selected: {},
}))((props) => <Tab disableRipple {...props} />);
const useStyles = makeStyles((theme) => ({
  root: {},
  row: {
    marginTop: 15,
  },
  spacer: {
    flexGrow: 1,
  },
  importButton: {
    marginRight: 15,
  },
  exportButton: {
    marginRight: 15,
  },
  searchInput: {
    marginRight: 15,
  },
  formControl: {
    margin: 25,
    fullWidth: true,
    display: "flex",
    wrap: "nowrap",
  },
  date: {
    marginTop: 3,
  },
  range: {
    marginTop: 13,
  },
}));

const Toolbar = (props) => {
  const { className, statusFilter, setStatusFilter, tabs, ...rest } = props;
  const [tabValue, setTabValue] = React.useState(statusFilter);

  const classes = useStyles();

  const handleChangeTab = (e, value) => {
    setTabValue(value);
    setStatusFilter(value);
  };

  return (
    <div {...rest} className={className}>
      <Grid container spacing={4}>
        <Grid item lg={3} sm={6} xl={3} xs={12} m={2}>
          <div className={classes}>
            <AntTabs
              value={tabValue}
              onChange={(e, value) => {
                handleChangeTab(e, value);
              }}
              aria-label="ant example"
            >
              {tabs.map((item) => (
                <AntTab key={item} label={item} />
              ))}
            </AntTabs>
            <Typography className={classes.padding} />
          </div>
        </Grid>
      </Grid>
    </div>
  );
};

Toolbar.propTypes = {
  className: PropTypes.string,
};

export default Toolbar;
```

Modifiquemos el archivo `src/pages/DashboardPage`:

```javascript
import React from "react";
import { makeStyles } from "@material-ui/styles";

+ import Toolbar from "../components/Toolbar.js";
import Table from "../components/Table.js";

const useStyles = makeStyles(theme => ({
  root: {
    padding: 15
  },
  content: {
    marginTop: 15
  },
}));

const DashboardPage = () => {
  const classes = useStyles();
+  const tabs = ['All', 'Shipped', 'Processing', 'Completed'];
+  const [statusFilter, setStatusFilter] = React.useState(0);

  const query = {
    "timeDimensions": [
      {
        "dimension": "Orders.createdAt",
        "granularity": "day"
      }
    ],
    "dimensions": [
      "Users.id",
      "Orders.id",
      "Orders.size",
      "Users.fullName",
      "Users.city",
      "Orders.price",
      "Orders.status",
      "Orders.createdAt"
    ],
+    "filters": [
+      {
+        "dimension": "Orders.status",
+        "operator": tabs[statusFilter] !== 'All' ? "equals" : "set",
+        "values": [
+          `${tabs[statusFilter].toLowerCase()}`
+        ]
+      }
+    ]
  };

  return (
    <div className={classes.root}>
+      <Toolbar
+        statusFilter={statusFilter}
+        setStatusFilter={setStatusFilter}
+        tabs={tabs}
+      />
      <div className={classes.content}>
        <Table
          query={query}/>
      </div>
    </div>
  );
};

export default DashboardPage;
```

¡Perfecto! Ahora la tabla de datos tiene un filtro que cambia entre diferentes tipos de órdenes:

![](/images/6.png)

Sin embargo, los pedidos tienen otros parámetros como precio y fechas. Creemos filtros para estos parámetros. Para hacerlo, modifique el archivo `src/components/Toolbar.js`:

```javascript
import "date-fns";
import React from "react";
import PropTypes from "prop-types";
import clsx from "clsx";
import { makeStyles } from "@material-ui/styles";
import Grid from "@material-ui/core/Grid";
import Typography from "@material-ui/core/Typography";
import Tab from "@material-ui/core/Tab";
import Tabs from "@material-ui/core/Tabs";
import withStyles from "@material-ui/core/styles/withStyles";
+ import DateFnsUtils from "@date-io/date-fns";
+ import {
+   MuiPickersUtilsProvider,
+   KeyboardDatePicker
+ } from "@material-ui/pickers";
+ import Slider from "@material-ui/core/Slider";

// ...

const Toolbar = props => {
  const { className,
+   startDate,
+   setStartDate,
+   finishDate,
+   setFinishDate,
+   priceFilter,
+   setPriceFilter,
    statusFilter,
    setStatusFilter,
    tabs,
    ...rest } = props;
  const [tabValue, setTabValue] = React.useState(statusFilter);
+ const [rangeValue, rangeSetValue] = React.useState(priceFilter);

  const classes = useStyles();

  const handleChangeTab = (e, value) => {
    setTabValue(value);
    setStatusFilter(value);
  };
+  const handleDateChange = (date) => {
+    setStartDate(date);
+  };
+  const handleDateChangeFinish = (date) => {
+    setFinishDate(date);
+  };
+ const handleChangeRange = (event, newValue) => {
+   rangeSetValue(newValue);
+ };
+ const setRangeFilter = (event, newValue) => {
+   setPriceFilter(newValue);
+ };

  return (
    <div
      {...rest}
      className={clsx(classes.root, className)}
    >
      <Grid container spacing={4}>
        <Grid
          item
          lg={3}
          sm={6}
          xl={3}
          xs={12}
          m={2}
        >
          <div className={classes}>
            <AntTabs value={tabValue} onChange={(e,value) => {handleChangeTab(e,value)}} aria-label="ant example">
              {tabs.map((item) => (<AntTab key={item} label={item} />))}
            </AntTabs>
            <Typography className={classes.padding} />
          </div>
        </Grid>
+        <Grid
+          className={classes.date}
+          item
+          lg={3}
+          sm={6}
+          xl={3}
+          xs={12}
+          m={2}
+        >
+          <MuiPickersUtilsProvider utils={DateFnsUtils}>
+            <Grid container justify="space-around">
+              <KeyboardDatePicker
+                id="date-picker-dialog"
+               label={<span style={{opacity: 0.6}}>Start Date</span>}
+                format="MM/dd/yyyy"
+                value={startDate}
+                onChange={handleDateChange}
+                KeyboardButtonProps={{
+                  "aria-label": "change date"
+                }}
+              />
+            </Grid>
+          </MuiPickersUtilsProvider>
+        </Grid>
+        <Grid
+          className={classes.date}
+          item
+          lg={3}
+          sm={6}
+          xl={3}
+          xs={12}
+          m={2}
+        >
+          <MuiPickersUtilsProvider utils={DateFnsUtils}>
+            <Grid container justify="space-around">
+              <KeyboardDatePicker
+                id="date-picker-dialog-finish"
+                label={<span style={{opacity: 0.6}}>Finish Date</span>}
+                format="MM/dd/yyyy"
+                value={finishDate}
+                onChange={handleDateChangeFinish}
+                KeyboardButtonProps={{
+                  "aria-label": "change date"
+                }}
+              />
+            </Grid>
+          </MuiPickersUtilsProvider>
+        </Grid>
+        <Grid
+          className={classes.range}
+          item
+          lg={3}
+          sm={6}
+          xl={3}
+          xs={12}
+          m={2}
+        >
+          <Typography id="range-slider">
+            Order price range
+          </Typography>
+          <Slider
+            value={rangeValue}
+            onChange={handleChangeRange}
+            onChangeCommitted={setRangeFilter}
+            aria-labelledby="range-slider"
+            valueLabelDisplay="auto"
+            min={0}
+            max={2000}
+          />
+        </Grid>
      </Grid>
    </div>
  );
};

Toolbar.propTypes = {
  className: PropTypes.string
};

export default Toolbar;
```

Para que estos filtros funcionen, necesitamos conectarlos al componente principal: agregar estado, modificar nuestra consulta y agregar nuevos props al componente `<Toolbar />`. Además, agregaremos ordenación a la tabla de datos. Entonces, modifique el archivo `src/pages/DashboardPage.js` de esta manera:

```javascript
// ...

const DashboardPage = () => {
  const classes = useStyles();
  const tabs = ['All', 'Shipped', 'Processing', 'Completed'];
  const [statusFilter, setStatusFilter] = React.useState(0);
+ const [startDate, setStartDate] = React.useState(new Date("2019-01-01T00:00:00"));
+ const [finishDate, setFinishDate] = React.useState(new Date("2022-01-01T00:00:00"));
+ const [priceFilter, setPriceFilter] = React.useState([0, 200]);
+ const [sorting, setSorting] = React.useState(['Orders.createdAt', 'desc']);

  const query = {
    timeDimensions: [
      {
        "dimension": "Orders.createdAt",
+    "dateRange": [startDate, finishDate],
        "granularity": "day"
      }
    ],
+    order: {
+      [`${sorting[0]}`]: sorting[1]
+    },
    "dimensions": [
      "Users.id",
      "Orders.id",
      "Orders.size",
      "Users.fullName",
      "Users.city",
      "Orders.price",
      "Orders.status",
      "Orders.createdAt"
    ],
    "filters": [
      {
        "dimension": "Orders.status",
        "operator": tabs[statusFilter] !== 'All' ? "equals" : "set",
        "values": [
          `${tabs[statusFilter].toLowerCase()}`
        ]
      },
+     {
+        "dimension": "Orders.price",
+        "operator": "gt",
+        "values": [
+         `${priceFilter[0]}`
+       ]
+     },
+     {
+       "dimension": "Orders.price",
+       "operator": "lt",
+       "values": [
+         `${priceFilter[1]}`
+       ]
+     },
    ]
  };

  return (
    <div className={classes.root}>
      <Toolbar
+       startDate={startDate}
+       setStartDate={setStartDate}
+       finishDate={finishDate}
+       setFinishDate={setFinishDate}
+       priceFilter={priceFilter}
+       setPriceFilter={setPriceFilter}
        statusFilter={statusFilter}
        setStatusFilter={setStatusFilter}
        tabs={tabs}
      />
      <div className={classes.content}>
        <Table
+          sorting={sorting}
+          setSorting={setSorting}
          query={query}/>
      </div>
    </div>
  );
};

export default DataTablePage;
```

¡Fantástico! Hemos agregado algunos filtros útiles. De hecho, puede agregar aún más filtros con lógica personalizada. Consulte la [documentación](https://cube.dev/docs/query-format?utm_source=dev-to&utm_medium=post&utm_campaign=react-data-table#filters-format) para conocer las opciones de formato de filtro.

¡Y hay una cosa más! Hemos agregado props de clasificación a la barra de herramientas, pero también debemos pasarlos al componente `<Table />`. Para solucionar esto, modifiquemos el archivo `src/components/Table.js`:

```javascript
// ...

+ import KeyboardArrowUpIcon from "@material-ui/icons/KeyboardArrowUp";
+ import KeyboardArrowDownIcon from "@material-ui/icons/KeyboardArrowDown";
import { useCubeQuery } from "@cubejs-client/react";
import CircularProgress from "@material-ui/core/CircularProgress";

// ...

const useStyles = makeStyles(theme => ({
  // ...
  actions: {
    justifyContent: "flex-end"
  },
+ tableRow: {
+   padding: '0 5px',
+   cursor: "pointer",
+   '.MuiTableRow-root.MuiTableRow-hover&:hover': {
+   }
+ },
+ hoverable: {
+   "&:hover": {
+     cursor: `pointer`
+   }
+ },
+ arrow: {
+   fontSize: 10,
+   position: "absolute"
+ }
}));

const statusColors = {
  completed: "success",
  processing: "info",
  shipped: "danger"
};

const TableComponent = props => {
-  const { className, query, cubejsApi, ...rest } = props;
+  const { className, sorting, setSorting, query, cubejsApi, ...rest } = props;

// ...

  if (resultSet) {

//...

+     const handleSetSorting = str => {
+       setSorting([str, sorting[1] === "desc" ? "asc" : "desc"]);
+     };

    return (
                            // ...

                <TableHead className={classes.head}>
                  <TableRow>
                    {tableHeaders.map((item) => (
                      <TableCell key={item.value + Math.random()} className={classes.hoverable}
+                                 onClick={() => {
+                                 handleSetSorting(`${item.value}`);
+                                 }}
                      >
                        <span>{item.text}</span>
+                        <Typography
+                          className={classes.arrow}
+                          variant="body2"
+                          component="span"
+                        >
+                          {(sorting[0] === item.value) ? (sorting[1] === "desc" ? <KeyboardArrowUpIcon/> :
+                            <KeyboardArrowDownIcon/>) : null}
+                        </Typography>
                      </TableCell>
                    ))}
                  </TableRow>
                </TableHead>
                         // ...

```

¡Maravilloso! Ahora tenemos la tabla de datos que admite completamente el filtrado y la clasificación:

![](/images/7.png)

¡Y eso es todo! ¡Felicitaciones por completar este tutorial!

Además, consulte el [código fuente completo](https://github.com/cube-js/cube.js/tree/master/examples/react-data-table) disponible en GitHub.

Ahora debería poder crear tablas de datos personalizadas impulsadas por Cube.js con React y Material UI para mostrar literalmente cualquier cantidad de datos en sus aplicaciones.

No dude en explorar [otros ejemplos]()https://github.com/cube-js/cube.js/#examples de lo que se puede hacer con Cube.js, como la [Guía del panel en tiempo real](https://real-time-dashboard.cube.dev/?utm_source=dev-to&utm_medium=post&utm_campaign=react-data-table) y la [Guía de la plataforma de análisis web de código abierto](https://web-analytics.cube.dev/?utm_source=dev-to&utm_medium=post&utm_campaign=react-data-table).
