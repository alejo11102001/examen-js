El repositorio debe contener un archivo README con instrucciones detalladas sobre el
proyecto, además de la información del coder (Nombre, Clan, correo, documento de
identidad). Este archivo debe detallar paso a paso cómo levantar y usar la solución,
garantizando que el Team Leader no tenga que realizar ingeniería inversa o adivinar cómo el
proyecto se corre.
https://github.com/Andres-Uribe55/crudClinic


const tblClientBody = document.getElementById('tblClientsBody');
const formEditClient = document.getElementById('frmEditClient');

document.addEventListener('DOMContentLoaded', () => {
    const form = document.getElementById('frmClient');
    const msg  = document.getElementById('msg');

    if (!form) {
        console.error('No existe #frmClient en el DOM');
        return;
    }

    form.addEventListener('submit', async (e) => {
        e.preventDefault();
        if (msg) msg.innerHTML = '';

        const formData = new FormData(form);
        const payload = {
        name:  formData.get('name')?.trim(),
        number_identification: formData.get('number_identification')?.trim(),
        address:  formData.get('address')?.trim(),
        phone: formData.get('phone')?.trim(),
        email: formData.get('email')?.trim()
        
        };

        if (!payload.name || !payload.number_identification || !payload.address || !payload.phone || !payload.email) {
        if (msg) msg.innerHTML = `<div class="alert alert-danger">Completa todos los campos.</div>`;
        return;
        }

        try {
        const res = await fetch(APP_URL + '/users', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
        });

        if (!res.ok) {
            const text = await res.t
        }

        const data = await res.json().catch(()=> ({}));
        if (msg) msg.innerHTML = `<div class="alert alert-success">cliente creado</div>`;
        form.reext().catch(()=>'');
            throw new Error(`HTTP ${res.status} ${text}`);set();

        } catch (err) {
        console.error('Error en POST:', err);
        if (msg) msg.innerHTML = `<div class="alert alert-danger">Error al crear cliente: ${err.message}</div>`;
        }
    });
});

// Nueva función para recargar la tabla (la usará la IIFE y también update/delete)
async function reloadClients() {
    const res = await fetch(APP_URL + '/users');
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const data = await res.json();
    const rows = Array.isArray(data) ? data : (data.rows || []);

    if (!tblClientBody) return;
    tblClientBody.innerHTML = '';
    rows.forEach(client => {
        tblClientBody.innerHTML += `
        <tr>
            <td>${client.id}</td>
            <td>${client.name}</td>
            <td>${client.identification}</td>
            <td>${client.address}</td>
            <td>${client.phone}</td>
            <td>${client.email}</td>
            <td class="text-end">
            <button class="btn btn-sm btn-primary" data-action="edit" data-id="${client.id}" data-name="${client.name}">Editar</button>
            <button class="btn btn-sm btn-danger"  data-action="delete" data-id="${client.id}" data-name="${client.name}">Eliminar</button>
            </td>
        </tr>
        `;
    });
}

    //Mantienes tu IIFE auto-invocada para cargar al abrir
(async function index() {
    try {
        await reloadClients();   // solo llamamos la función auxiliar
    } catch (error) {
        console.error('Error en GET:', error);
    }
})();

// === API: solo fetch, sin tocar el DOM ===
async function updateClients(id, payload) {
    const res = await fetch(`${APP_URL}/users/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
    });
    if (!res.ok) {
        const text = await res.text().catch(() => '');
        throw new Error(`HTTP ${res.status} ${text}`);
    }
    // devuelve el cliente actualizado
    return res.json().catch(() => ({}));
}

// DOM: solo orquesta lectura del form, llamada a la API y UI ===
window.addEventListener('DOMContentLoaded', () => {
    const frm = document.getElementById('frmEditClient');

    frm.addEventListener('submit', async (e) => {
        e.preventDefault();

        try {
        const id    = document.getElementById('clientId').value;
        const name  = document.getElementById('clientName').value.trim();
        const identification = document.getElementById('clientIdentification').value.trim();
        const address  = document.getElementById('clientAddress').value.trim();
        const phone = document.getElementById('clientPhone').value.trim();
        const email = document.getElementById('clientEmail').value.trim();

        await updatePatient(id, { name, identification, address, phone, email });

        // Cerrar modal
        bootstrap.Modal.getOrCreateInstance(
            document.getElementById('exampleModalClient')
        ).hide();

        // Refrescar tabla y avisar
        await reloadClients();
        alert('CLlente actualizado');

        } catch (err) {
        console.error('Error en PUT:', err);
        alert('No se pudo actualizar: ' + err.message);
        }
    });
});

// --- DELETE ---
async function deleteClient(id, name) {
    try {
        // usar el nombre que viene en el botón
        let nameLabel = name ? `"${name}"` : `#${id}`;

        // confirmar
        const ok = confirm(`¿Eliminar Cliente ${nameLabel}?`);
        if (!ok) return;

        // petición DELETE
        const res = await fetch(`${APP_URL}/users/${id}`, { method: 'DELETE' });
        if (!res.ok) {
        const t = await res.text().catch(() => '');
        throw new Error(`HTTP ${res.status} ${t}`);
        }

        await reloadClients();                  // refresca tabla
        // aviso
        alert(`Cliente ${nameLabel} eliminado`);

    } catch (error) {
        console.error('Error en DELETE:', error);
        alert('No se pudo eliminar: ' + error.message);
    }
}

// --- Delegación de eventos SOLO en la tabla de pacientes ---
if (tblClientBody) {
    tblClientBody.addEventListener('click', (e) => {
        const btn = e.target.closest('button.btn-sm');
        if (!btn) return;

        const id = btn.dataset.id;
        const name = btn.dataset.name;
        const action = btn.dataset.action;
        if (!id || !action) return;

        if (action === 'edit') {
            showClient(id);
        } else if (action === 'delete') {
            deleteClient(id, name);
        }
    });
}


async function showClient(id){
    try {
        const res = await fetch(`${APP_URL}/users/${id}`);
        if (!res.ok) {
            throw new Error(`Error HTTP ${res.status} al buscar el cliente.`);
        }
        const client = await res.json(); // client es un array: [{...}]

        console.log('Datos recibidos de la API:', client);

        // VOLVEMOS A USAR [0] PORQUE LA API DEVUELVE UN ARRAY

        document.getElementById('clientId').value = client[0].id;
        document.getElementById('clientName').value = client[0].name;
        document.getElementById('clientIdentification').value = client[0].identification;
        document.getElementById('clientAddress').value = client[0].address;
        document.getElementById('clientPhone').value = client[0].phone;
        document.getElementById('clientEmail').value = client[0].email;

        // Mostrar el modal
        const modalElement = document.getElementById('exampleModalClient');
        const modal = bootstrap.Modal.getOrCreateInstance(modalElement);
        modal.show();

    } catch (error) {
        console.error('Error en showClient:', error);
        alert('No se pudieron cargar los datos del cliente.');
    }
}








<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8" />
<title>Nuevo Cliente</title>
<meta name="viewport" content="width=device-width, initial-scale=1" />
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
<div class="container py-4">
<h1 class="h4 mb-3">Crear Cliente</h1>

<form id="frmClient" class="card p-3 shadow-sm bg-white">
    <div class="row g-3">
        <div class="mb-3">
            <label for="clientName" class="form-label">Nombre</label>
            <input type="text" class="form-control" id="clientName" name="name" required>
        </div>
        <div class="mb-3">
            <label for="clientIdentification" class="form-label">Identificación</label>
            <input type="text" class="form-control" id="lientIdentification" name="identification" required>
        </div>
        <div class="mb-3">
            <label for="clientAddress" class="form-label">Dirección</label>
            <input type="text" class="form-control" id="clientAddress" name="address" required>
        </div>
        <div class="mb-3">
            <label for="clientPhone" class="form-label">Telefono</label>
            <input type="text" class="form-control" id="clientPhone" name="phone" required>
        </div>
        <div class="mb-3">
            <label for="clientEmail" class="form-label">Correo</label>
            <input type="email" class="form-control" id="clientEmail" name="email" required>
        </div>
    </div>

    <div class="mt-3 d-flex gap-2">
    <button class="btn btn-primary" type="submit">Guardar</button>
    <button class="btn btn-outline-secondary" type="reset">Limpiar</button>
    </div>

    <div id="msg" class="mt-3"></div>
</form>
<div class="mt-3">
    <a class="btn btn-link p-0" href="./home.html">Volver</a>
</div>
</div>

<script src="./js/config.js"></script>
<script src="./js/scripts.js"></script>
</body>
</html>
const express = require('express');
const { Pool } = require('pg');
const cors = require('cors');

const app = express();
app.use(express.json());
app.use(cors())

// Configura tu conexión a PostgreSQL
const pool = new Pool({
    user: 'root',
    host: '168.119.183.3',
    database: 'pd_diego_zuluaga_vanrossum',
    password: 's7cq453mt2jnicTaQXKT',
    port: 5432
});

//USERS

app.get("/users", async (req, res) => {
    try {
        let result = await pool.query("SELECT * FROM clients order by id asc");
        return res.json(result);
    } catch (error) {
        res.status(500).json({error : 'error'})
    }
});

app.get("/users/:id", async (req, res) => {
    try {
        let result = await pool.query("SELECT * FROM clients WHERE id = $1", [req.params.id]);
        return res.json(result.rows);
    } catch (error) {
        res.status(500).json({error : 'error'})
    }
});

app.post('/users', async (req, res) => {
    const { name, number_identification, address, phone, email } = req.body;
    try {
        const result = await pool.query(
        'INSERT INTO clients (name, number_identification, address, phone, email) VALUES ($1, $2, $3, $4, $5) RETURNING *',
        [name, number_identification, address, phone, email]
        );
        res.status(201).json(result.rows[0]);
    } catch (error) {
        res.status(500).json({ error: 'Error al crear el cliente' });
    }
});

app.put('/users/:id', async (req, res) => {
    const { id } = req.params;
    const { name, number_identification, address, phone, email } = req.body;
    try {
        const result = await pool.query(
            'UPDATE clients SET name = $1, number_identification = $2, address = $3, phone = $4, email = $5 WHERE id = $6 RETURNING *',
            [name, number_identification, address, phone, email, id]
        );
        if (result.rowCount === 0) {
            return res.status(404).json({ error: 'cliente no encontrado' });
        }
        res.json({ message: 'cliente actualizado' });
    } catch (error) {
        res.status(500).json({ error: 'Error al actualizar el cliente' });
    }
});

app.delete('/users/:id', async (req, res) => {
    const { id } = req.params;
    try {
        const result = await pool.query('DELETE FROM clients WHERE id = $1 RETURNING *', [id]);
        if (result.rows.length === 0) return res.status(404).json({ error: 'cliente no encontrado' });
        res.json({ mensaje: 'cliente eliminado' });
    } catch (error) {
        res.status(500).json({ error: 'Error al eliminar el cliente' });
    }
});

app.listen(3000, () => {
    console.log('Servidor corriendo en http://localhost:3000');
});
