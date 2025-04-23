# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

SELECT  cli.id_cliente, cli.nombre, COUNT(ca.num_cuenta) AS cantidad_cuenta, SUM(ca.saldo) AS saldo_total
FROM  Cliente cli
JOIN Cuenta ca ON cli.id_cliente = ca.id_cliente
GROUP BY cli.id_cliente, cli.nombre
HAVING COUNT(ca.num_cuenta) > 1
ORDER BY saldo_total DESC;

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

SELECT cli.id_cliente, cli.nombre,  
SUM(CASE WHEN trx.tipo_transaccion = 'deposito' THEN trx.monto END), 0 AS total_depositos,
SUM(CASE WHEN trx.tipo_transaccion = 'retiro' THEN trx.monto END), 0  AS total_retiros
FROM Cliente cli
INNER JOIN Cuenta ca ON cli.id_cliente = ca.id_cliente
INNER JOIN Transaccion trx ON ca.num_cuenta = trx.num_cuenta
GROUP BY cli.id_cliente, cli.nombre  
ORDER BY total_depositos DESC, total_retiros DESC;

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

SELECT ca.num_cuenta, ca.tipo_cuenta
FROM Cuenta ca
LEFT JOIN Tarjeta tarj ON ca.num_cuenta = tarj.num_cuenta
WHERE tarj.num_cuenta IS NULL;

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

SELECT ca.tipo_cuenta,
AVG(ca.saldo) AS saldo_promedio
FROM Cuenta ca
WHERE EXISTS (
SELECT 1 FROM Transaccion trx WHERE trx.num_cuenta = ca.num_cuenta AND trx.fecha >= CURRENT_DATE - INTERVAL '30 days')
GROUP BY ca.tipo_cuenta;

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

SELECT DISTINCT cli.id_cliente, cli.nombre
FROM Cliente cli
INNER JOIN Cuenta ca ON cli.id_cliente = ca.id_cliente
INNER JOIN Transaccion trx ON ca.num_cuenta = trx.num_cuenta
INNER JOIN Transferencia transf ON trx.id_transaccion = transf.id_transaccion
WHERE NOT EXISTS (SELECT 1 FROM Transaccion trxn 
INNER JOIN Retiro ret ON trxn.id_transaccion = ret.id_transaccion
WHERE trxn.num_cuenta = ca.num_cuenta AND LOWER(ret.canal) = 'cajero'
);
