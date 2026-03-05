# 🏦 MoneyBank API Microservice

Este proyecto implementa un microservicio transaccional bancario robusto utilizando **.NET Core** y **Clean Architecture**. El sistema gestiona el ciclo de vida de cuentas bancarias (Ahorros y Corrientes) y procesa transacciones financieras (Depósitos y Retiros) aplicando reglas de negocio complejas, como validación de sobregiros y control de fondos.

## 🛠️ Tecnologías

- **Framework**: .NET 6+ (C#)
- **Base de Datos**: MySQL
- **ORM**: Entity Framework Core
- **Arquitectura**: Clean Architecture (DTOs, Services, Models, Exception Middleware)
- **Contenedores**: Docker (para API y Base de Datos)

## 🚀 Configuración e Instalación

### 1. Base de Datos

El proyecto requiere una instancia de MySQL. Ejecute los scripts ubicados en la carpeta de base de datos en el siguiente orden para preparar el entorno:

1.  `01_Create_Database.sql`
2.  `02_Create_User.sql`
3.  `03_TAB_Accounts.sql`
4.  `04_INS_Acounts.sql`

### 2. Ejecución

Asegúrese de configurar la cadena de conexión en `appsettings.json` y ejecute el proyecto:

```bash
dotnet run --project MoneyBankAPI
```

Tenga en cuenta complementarlo con las Anotaciones Necesarias para manejar los conceptos de :

- Llave ([Key])
- Requeridos (ej. El campo Nombre del Propietario es Requerido) ([Required])
- Longitud (ej. El campo Numero de La Cuenta tiene una longitud maxima de 10 caracteres) ([MaxLength])
- Valores (ej. El campo Numero de la Cuenta Solo Acepta Numeros) ([RegularExpression("\d{10}")])
- Monedas (ej, El campo Balance debe ser en formato Moneda (0.00)) ([RegularExpression("^\d+.?\d{0,2}$")])
- Lista de Datos (ej. El campo Tipo de Cuenta solo permite (A o C)) ([RegularExpression("[AC]")])
- Tipos ([DataType(DataType.Date)])

Para mas informacion consulte [aqui](https://www.bytehide.com/blog/data-annotations-in-csharp)

# Object Relational Mapping (ORM) - Entity Framework

Recuerde adicionar los Paquetes (Nugets) necesarios para acceder a la base de datos:

- Microsoft.EntityFrameworkCore
- Microsoft.EntityFrameworkCore.Design
- Microsoft.EntityFrameworkCore.Tools
- MySql.EntityFrameworkCore (asegurese de que sea esta y no otra)

# Controlador

Asegure que el Nuevo controlador El controlador **AccountsController** de tipo **API**, adicionando Acciones de Entity Framework, utilizando el Modelo de **Account** y el contexto de **AppDbContext**.

# EndPoints

Al seleccionar el Controlador con acciones que usan Entity Framework se crean las acciones principales de CRUD para las cuentas, en este punto ya es posible probrar estas acciones y confirmar su correcta ejecucion.

## GetAccounts

### Metodo

GET /api/Accounts

### Request

Sin Contenido

### Response

Arreglo con los datos de las cuentas

```json
[
  {
    "id": 0,
    "accountType": "C",
    "creationDate": "2023-10-04",
    "accountNumber": "6",
    "ownerName": "string",
    "balanceAmount": 0,
    "overdraftAmount": 0
  }
]
```

## GetAccounts

Utilice el Modificador [FromQuery] para Pasar el Numero de la Cuenta

### Metodo

GET /api/Accounts?AccountNumber={accountNumber}

### Request

Se envia el **accountNumber** o Numero de la Cuenta

### Response

```csharp
[
  {
    "id": 1,
    "accountType": "C",
    "creationDate": "2023-06-30T00:00:00",
    "accountNumber": "3016892501",
    "ownerName": "Yurley Orejuela Ramirez",
    "balanceAmount": 1500000,
    "overdraftAmount": 0
  }
]
```

## GetAccount

### Metodo

GET /api/Accounts/{id}

### Request

Se envia el **Id**

### Response

```csharp
{
  "id": 1,
  "accountType": "C",
  "creationDate": "2023-06-30T00:00:00",
  "accountNumber": "3016892501",
  "ownerName": "Yurley Orejuela Ramirez",
  "balanceAmount": 1500000,
  "overdraftAmount": 0
}
```

## PostAccount

### Metodo

POST /api/Accounts

### Request

```csharp
{
  "id": 0,
  "accountType": "A",
  "creationDate": "2023-10-04",
  "accountNumber": "6087523149",
  "ownerName": "John Doe",
  "balanceAmount": 1000000,
  "overdraftAmount": 0
}
```

### Response

```csharp
{
  "id": 4,
  "accountType": "A",
  "creationDate": "2023-10-04",
  "accountNumber": "6087523149",
  "ownerName": "John Doe",
  "balanceAmount": 1000000,
  "overdraftAmount": 0
}
```

## PutAccount

### Metodo

PUT /api/Accounts/{id}

### Request

Se envia el **Id** y el body con el contenido a modificar.

```csharp
{
  "id": 4,
  "accountType": "A",
  "creationDate": "2023-10-04",
  "accountNumber": "6087523149",
  "ownerName": "John Alexander Doe",
  "balanceAmount": 1000000,
  "overdraftAmount": 0
}
```

### Response

Sin contenido

## DeleteAccount

### Metodo

DELETE /api/Accounts/{id}

### Request

Se envia el **Id**

### Response

Sin contenido

# Acciones de Cajero

```csharp
public class Transaction
{
    public int Id { get; set; }
    public string AccountNumber { get; set; } = null!;
    public decimal ValueAmount { get; set; }
}
```

## Deposito

### Metodo

PUT /api/Accounts/{id}/Deposit

### Request

```json
{
  "id": 0,
  "accountNumber": "2427744115",
  "valueAmount": 0
}
```

## Retiro

### Metodo

PUT /api/Accounts/{id}/Withdrawal

### Request

```json
{
  "id": 0,
  "accountNumber": "2427744115",
  "valueAmount": 0
}
```

# Ejemplos

A continuación algunos ejemplos de la Logica de Deposito y Retiro para los tipos de cuentas.

## Cuenta de Ahorros / Deposito

Balance Actual = $200,000.00
Sobregiro= $0.00

Deposito = $500,000.00

Nuevo Balance = $700,000.00
Sobregiro = $0.00

#### Regla

```csharp
Balance += Deposit
```

### Cuenta de Ahorros / Retiro

Balance Actual = $700,000.00
Sobregiro= $0.00

Retiro = $300,000.00

Nuevo Balance = $400,000.00
Sobregiro = $0.00

#### Regla

```csharp
if (Withdrawal <= Balance)
{
    Balance -= Withdrawal
}
else
{
    "Fondos Insuficientes"
}
```

### Cuenta Corriente / Deposito

Balance Actual = $300,000.00
Sobregiro= $700,000.00

Deposito = $500,000.00

Nuevo Balance = $800,000.00
Sobregiro = $200,000.00

#### Regla

```csharp
Balance += Deposit

if ( Overdraft > 0 && Balance < MAX_OVERDRAFT)
{
    OverDraft = MAX_OVERDRAFT - Balance
}
else
{
    OverDraft = 0
}
```

### Cuenta Corriente / Retiro

Balance Actual = $800,000.00
Sobregiro= $200,000.00

Retiro = $500,000.00

Nuevo Balance = $300,000.00
Sobregiro = $700,000.00

#### Reglas

```csharp
if (Withdrawal <= Balance)
{
    Balance -= Withdrawal

    if( Overdraft > 0 && Balance < MAX_OVERDRAFT)
    {
        OverDraft = MAX_OVERDRAFT - Balance
    }
}
else
{
    "Fondos Insuficientes"
}
```
