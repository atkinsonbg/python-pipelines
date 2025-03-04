# Relational

## CREATE TABLES
```
CREATE TABLE Tenants (
    TenantID          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    Name             VARCHAR(255) NOT NULL, -- Customer name i.e. CareSource, Highmark
    TimeZone         VARCHAR(50),
    UI_Settings      JSONB DEFAULT '{}'  -- Stores UI preferences, this assumes the ui settings are the same for all instances
);
```

```
CREATE TABLE Products (
    ProductID       UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    Name           VARCHAR(255) NOT NULL UNIQUE,  -- CareWebQI, Indicia
    Description    TEXT
);
```

```--This is instance of a product which aligns 1 to 1 with Salesforce LMRs
CREATE TABLE Tenant_Products (
    TenantProductID UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    TenantID       UUID NOT NULL REFERENCES Tenants(TenantID) ON DELETE CASCADE,
    ProductID      UUID NOT NULL REFERENCES Products(ProductID) ON DELETE CASCADE,
    InstanceIdetifier  VARCHAR(255) NOT NULL,  -- this is the identifier(database name) used to get the config (caresoure.carewebqi.com, hhprod.carewebqi.com, etc....) may or may not have ".carewebqi.com" on it.
    LicensedAt     TIMESTAMP DEFAULT NOW(), 
    LicenseDetails JSONB DEFAULT '{}' -- the licence information for that instance of a tenantProduct
);
```

## SELECTS
```
SELECT P.Name
FROM Tenant_Products TP
JOIN Products P ON TP.ProductID = P.ProductID
WHERE TP.TenantID = 'T1';
```

```
SELECT E.Name AS Environment, TPC.ConfigSettings
FROM Tenant_Product_Configs TPC
JOIN Tenant_Products TP ON TPC.TenantProductID = TP.TenantProductID
JOIN Environments E ON TPC.EnvironmentID = E.EnvironmentID
WHERE TP.TenantID = 'T1' AND TP.ProductID = 'P1' AND E.Name = 'Prod';
```

```
SELECT T.Name AS Tenant, P.Name AS Product, E.Name AS Environment, TPC.ConfigSettings
FROM Tenant_Product_Configs TPC
JOIN Tenant_Products TP ON TPC.TenantProductID = TP.TenantProductID
JOIN Environments E ON TPC.EnvironmentID = E.EnvironmentID
JOIN Tenants T ON TP.TenantID = T.TenantID
JOIN Products P ON TP.ProductID = P.ProductID
WHERE T.TenantID = 'T1';
```

## INSERTS
```
INSERT INTO Tenants (TenantID, Name, TimeZone, UI_Settings)
VALUES
    ('T1', 'Tenant A', 'UTC', '{"theme": "light", "dashboard": "grid"}'),
    ('T2', 'Tenant B', 'EST', '{"theme": "dark", "dashboard": "list"}');
```

```
INSERT INTO Products (ProductID, Name, Description)
VALUES
    ('P1', 'Product A', 'SaaS product A description'),
    ('P2', 'Product B', 'SaaS product B description'),
    ('P3', 'Product C', 'SaaS product C description');
```

```
INSERT INTO Tenant_Products (TenantProductID, TenantID, ProductID, LicensedAt)
VALUES
    ('TP1', 'T1', 'P1', NOW()),  -- Tenant A licenses Product A
    ('TP2', 'T1', 'P2', NOW()),  -- Tenant A licenses Product B
    ('TP3', 'T2', 'P3', NOW());  -- Tenant B licenses Product C
```

```
INSERT INTO Environments (EnvironmentID, Name)
VALUES
    ('E1', 'Dev'),
    ('E2', 'Prod'),
    ('E3', 'Stage');
```

```
INSERT INTO Tenant_Product_Configs (ConfigID, TenantProductID, EnvironmentID, ConfigSettings)
VALUES
    -- Tenant A's Product A Configurations
    ('C1', 'TP1', 'E1', '{"api_limit": 100, "featureX": true}'),  -- Dev
    ('C2', 'TP1', 'E2', '{"api_limit": 200, "featureX": false}'), -- Prod

    -- Tenant A's Product B Configurations
    ('C3', 'TP2', 'E2', '{"theme": "dark", "enable_logging": true}'), -- Prod

    -- Tenant B's Product C Configurations
    ('C4', 'TP3', 'E3', '{"region": "us-east", "auto_scale": true}'), -- Stage
    ('C5', 'TP3', 'E2', '{"region": "us-west", "auto_scale": false}'); -- Prod
```

## VERIFY INSERTS
```
SELECT * FROM Tenants;

SELECT * FROM Products;

SELECT T.Name AS Tenant, P.Name AS Product, TP.LicensedAt
FROM Tenant_Products TP
JOIN Tenants T ON TP.TenantID = T.TenantID
JOIN Products P ON TP.ProductID = P.ProductID;

SELECT T.Name AS Tenant, P.Name AS Product, E.Name AS Environment, TPC.ConfigSettings
FROM Tenant_Product_Configs TPC
JOIN Tenant_Products TP ON TPC.TenantProductID = TP.TenantProductID
JOIN Environments E ON TPC.EnvironmentID = E.EnvironmentID
JOIN Tenants T ON TP.TenantID = T.TenantID
JOIN Products P ON TP.ProductID = P.ProductID;
```

# NoSQL
```
{
  "_id": "T1",
  "name": "Tenant A",
  "timeZone": "UTC",
  "uiSettings": {
    "theme": "light",
    "dashboard": "grid"
  },
  "licensedProducts": [
    {
      "productId": "P1",
      "name": "Product A",
      "environments": [
        {
          "name": "Dev",
          "configSettings": {
            "api_limit": 100,
            "featureX": true
          }
        },
        {
          "name": "Prod",
          "configSettings": {
            "api_limit": 200,
            "featureX": false
          }
        }
      ]
    },
    {
      "productId": "P2",
      "name": "Product B",
      "environments": [
        {
          "name": "Prod",
          "configSettings": {
            "theme": "dark",
            "enable_logging": true
          }
        }
      ]
    }
  ]
}
```

