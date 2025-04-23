# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.
db.clientes.aggregate([
  {$project: {
      nombre: 1,
      cantidad_cuentas: { $size: "$cuentas" },
      saldos: { $map: {
          input: "$cuentas",
          as: "cuenta",
          in: "$$cuenta.saldo"
        }}}},
  {
    $match: { "cantidad_cuentas": { $gt: 1 } }  
  },
  { $project: {
      nombre: 1,
      cantidad_cuentas: 1,
      saldo_total: { $sum: "$saldos" }
    }},
  {
    $sort: { saldo_total: -1 }
  }
])

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

db.transacciones.aggregate([
  { $group: {
      _id: { cliente_ref: "$cliente_ref", tipo_transaccion: "$tipo_transaccion" },
      cantidad_transacciones: { $sum: 1 },
      monto_total: { $sum: "$monto" }
    }
  },
  { $group: {
      _id: "$_id.cliente_ref",
      transacciones: { 
        $push: {
          tipo_transaccion: "$_id.tipo_transaccion",
          cantidad_transacciones: "$cantidad_transacciones",
          monto_total: "$monto_total"
        }
      }
    }
  }

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.
db.clientes.aggregate([
  { $unwind: "$cuentas" },
  { $unwind: "$cuentas.tarjetas" },
  {
    $match: { "cuentas.tarjetas.tipo_tarjeta": "credito" }
  },
  {$group: {
      _id: "$_id",
      nombre: { $first: "$nombre" },
      cedula: { $first: "$cedula" },
      tarjetas_credito: { $push: "$cuentas.tarjetas.numero_tarjeta" },
      cantidad_tarjetas_credito: { $sum: 1 }
    }
  },
  {
    $match: {
      cantidad_tarjetas_credito: { $gt: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      nombre: 1,
      cedula: 1,
      cantidad_tarjetas_credito: 1,
      tarjetas_credito: 1
    }
  }
])


## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

db.transacciones.aggregate([
  {
    $match: { tipo_transaccion: "deposito" } 
  },
  {$project: {
      mes: { $month: "$fecha" }, 
      anio: { $year: "$fecha" },
      medio_pago: "$detalles_deposito.medio_pago"  
    }
  },
  {
    $group: {
      _id: { mes: "$mes", anio: "$anio", medio_pago: "$medio_pago" },  
      cantidad: { $sum: 1 }
    }
  },
  {
    $sort: { "_id.anio": 1, "_id.mes": 1, "cantidad": -1 }  
  },
  {
    $project: {
      mes: "$_id.mes",
      anio: "$_id.anio",
      medio_pago: "$_id.medio_pago",
      cantidad: 1,
      _id: 0
    }
  }
])

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.


