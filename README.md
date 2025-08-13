El repositorio debe contener un archivo README con instrucciones detalladas sobre el
proyecto, además de la información del coder (Nombre, Clan, correo, documento de
identidad). Este archivo debe detallar paso a paso cómo levantar y usar la solución,
garantizando que el Team Leader no tenga que realizar ingeniería inversa o adivinar cómo el
proyecto se corre.

Consultas avanzadas (solo desde Postman)
1. Total pagado por cada cliente
"Como administrador del sistema, necesito saber cuánto ha pagado cada cliente en
total, para poder llevar un control de los ingresos y verificar los saldos generales."
2. Facturas pendientes con información de cliente y transacción asociada
"Como responsable financiero, necesito identificar las facturas que aún no han sido
pagadas completamente, junto con el nombre del cliente y la transacción
correspondiente, para gestionar el cobro o seguimiento."
3. Listado de transacciones por plataforma
"Como analista, necesito poder ver todas las transacciones hechas desde una
plataforma específica (como Nequi o Davipla

Incluir un README.md en inglés con:
• Descripción del sistema.
• Instrucciones para ejecutar el proyecto.
• Tecnologías utilizadas.
• Explicación de la normalización.
• Instrucciones para la carga masiva desde CSV.
• Explicación de las consultas avanzadas.
• Captura del modelo relacional.
• Datos del desarrollador (nombre, clan, correo).


// A) Total pagado por cada cliente
app.get('/reports/clients/total-paid', async (req, res) => {
  try {
    const q = `
      SELECT
        c.id AS client_id,
        c.name AS client_name,
        COALESCE(SUM(b.amount_paid), 0) AS total_paid
      FROM clients c
      LEFT JOIN billings b ON b.client_id = c.id
      GROUP BY c.id, c.name
      ORDER BY total_paid DESC, client_name ASC;
    `;
    const { rows } = await pool.query(q);
    res.json(rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// B) Facturas pendientes con info de cliente y última transacción asociada
// Definición: pendiente = su última transacción NO es 'Completada' (o no tiene transacciones)
app.get('/reports/billings/pending', async (req, res) => {
  try {
    const q = `
      WITH last_tx AS (
        SELECT DISTINCT ON (t.billing_id)
               t.billing_id, t.id AS transaction_id, t.status, t.platform, t.date_time
        FROM transactions t
        ORDER BY t.billing_id, t.date_time DESC
      )
      SELECT
        b.id AS billing_id,
        b.number_billing,
        b.period_billing,
        b.amount_paid,
        c.id AS client_id,
        c.name AS client_name,
        lt.transaction_id,
        lt.status AS transaction_status,
        lt.platform AS transaction_platform,
        lt.date_time AS transaction_date
      FROM billings b
      JOIN clients c ON c.id = b.client_id
      LEFT JOIN last_tx lt ON lt.billing_id = b.id
      WHERE COALESCE(lt.status, 'Pendiente') <> 'Completada'
      ORDER BY b.id;
    `;
    const { rows } = await pool.query(q);
    res.json(rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// C) Listado de transacciones por plataforma (Nequi | Daviplata)
app.get('/reports/transactions/by-platform/:platform', async (req, res) => {
  try {
    const { platform } = req.params; // 'Nequi' | 'Daviplata'
    const q = `
      SELECT
        t.id,
        t.date_time,
        t.status,
        t.type,
        t.platform,
        t.billing_id,
        b.number_billing,
        c.name AS client_name
      FROM transactions t
      JOIN billings b ON b.id = t.billing_id
      JOIN clients c ON c.id = b.client_id
      WHERE t.platform = $1
      ORDER BY t.date_time DESC;
    `;
    const { rows } = await pool.query(q, [platform]);
    res.json(rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

Ejemplos en Postman / cURL
Total por cliente
GET http://localhost:3000/reports/clients/total-paid

Facturas pendientes + cliente + última transacción
GET http://localhost:3000/reports/billings/pending

Transacciones por plataforma
GET http://localhost:3000/reports/transactions/by-platform/Nequi
GET http://localhost:3000/reports/transactions/by-platform/Daviplata
