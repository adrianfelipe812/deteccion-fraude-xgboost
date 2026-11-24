# Datos

| Archivo | Filas | Columnas | Separador |
|---|---|---|---|
| `train.csv` | 848.548 | 9 (incluye `Fraude`) | `;` |
| `test.csv` | 200.027 | 8 | `;` |
| `sample_submission.csv` | 200.027 | 2 (`ID`, `Fraude`) | `,` |

## Diccionario de variables

| Variable | Tipo | Descripción |
|---|---|---|
| `ID` | entero | Identificador secuencial del registro (1 a 1.048.574, repartido entre train y test) |
| `Num_Tarjetas` | entero | Número de tarjetas del cliente (1 o 2) |
| `Balance_Tj` | entero | Saldo de la tarjeta (0 a 39.725) |
| `Total_Transac` | entero | Total de transacciones (0 a 100) |
| `Trans_Int` | entero | Transacciones internacionales (0 a 60) |
| `Estado` | categórica | Estado de EE. UU. (51 valores) |
| `Genero` | categórica | `Male` / `Female` |
| `Tipo_Tarjeta` | categórica | American Express, MasterCard, Visa, Discover |
| `Fraude` | binaria | Variable objetivo (1 = fraude). Prevalencia: 5,985 % |

## Revisión de privacidad

Antes de publicar se revisó si hacía falta anonimizar los datos. **No fue necesario**:

- No hay identificadores directos: ni nombres, ni documentos, ni números de tarjeta, ni correos, ni direcciones, ni fechas.
- `ID` es un índice consecutivo del conjunto y no remite a ninguna persona ni cuenta real.
- Las variables restantes son agregados numéricos y tres categorías de baja cardinalidad (estado, género y franquicia).
- Los datos son de uso académico: el docente los entregó para una competencia de clase en Kaggle. Su estructura coincide con la del conjunto sintético público *ccFraud*, de fraude con tarjetas de crédito.

Por eso los archivos se publican tal como se usaron en la competencia, sin modificarlos.
