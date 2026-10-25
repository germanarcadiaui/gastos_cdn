# Control de Gastos CDN

App web para registrar, revisar y dar seguimiento a los gastos de CDN por Unidad de Negocio (UDN).
React + Vite, base de datos y archivos en Supabase, publicación en Netlify.

## Qué hace

- **Registro de gastos** en forma tabular, con filtro en cada columna, edición directa en la celda,
  total del importe filtrado y exportación a Excel (CSV).
- **Tres formas de capturar**: manual, desde un XML (llena proveedor, RFC, folio, fecha e importe)
  e **importación masiva de una carpeta de XML** (adjunta el PDF con el mismo nombre).
- **PDF factura, PDF pago y documentos de soporte** (escaneados, varios por gasto). Al pasar el
  mouse sobre un PDF se ve la vista previa; con clic se abre completo.
- **Catálogos** que se van llenando solos: tipos de gasto (con el área que los revisa),
  proveedores (con días de crédito) y descripciones.
- **Fecha de expiración** = fecha de emisión + días de crédito (los días se toman del proveedor
  si los tiene en el catálogo, o del XML, o se capturan).
- **Fecha de elaboración** y **usuario que registró**: automáticos y no modificables.
- **Revisiones**: Coordinador Administrativo o Flotillas (según el tipo de gasto) y al final
  Dirección. La celda se pone en verde al validar y muestra quién y cuándo.
- **Usuarios y UDN**: el super usuario da de alta usuarios, les asigna rol y las UDN que pueden ver.

## Roles

| Rol | Puede |
|---|---|
| Super usuario | Todo: UDN, usuarios, ve todas las UDN, cualquier revisión |
| Director | Capturar y marcar la revisión de Dirección (cuando el área ya validó) |
| Coordinador Administrativo | Capturar y validar los gastos cuyo tipo revisa Coordinación |
| Flotillas | Capturar y validar los gastos cuyo tipo revisa Flotillas |
| Capturista | Capturar y editar gastos de sus UDN |

Cuando Dirección ya revisó un gasto, solo se pueden cambiar el PDF de pago, el estatus de pago,
las observaciones y los documentos de soporte.

## Puesta en marcha

### 1. Supabase
1. Crea un proyecto nuevo en supabase.com.
2. **SQL Editor → New query**: pega todo `supabase/schema.sql` y ejecútalo.
3. **Authentication → Users → Add user**: correo y contraseña del super usuario
   (marca *Auto Confirm User*).
4. En el SQL Editor ejecuta, con tu correo:
   ```sql
   update public.perfiles set rol = 'superadmin', nombre = 'Tu nombre' where email = 'tu@correo.com';
   ```
5. **Project Settings → API**: copia la *Project URL*, la *anon key* y la *service_role key*.

### 2. Netlify
1. Sube esta carpeta a un repositorio de GitHub y conéctalo en Netlify
   (o usa `netlify deploy` desde la terminal).
2. En **Site configuration → Environment variables** agrega:
   - `VITE_SUPABASE_URL` y `SUPABASE_URL` → Project URL
   - `VITE_SUPABASE_ANON_KEY` → anon key
   - `SUPABASE_SERVICE_ROLE_KEY` → service_role key (solo aquí, nunca en el código)
3. Despliega. El build y la función de usuarios ya están configurados en `netlify.toml`.

### 3. Primer uso
1. Entra con el super usuario.
2. **UDN**: da de alta las unidades de negocio (código de hasta 4 caracteres + descripción).
3. **Catálogos → Tipos de gasto**: da de alta los gastos e indica quién los revisa.
4. **Usuarios**: crea los usuarios, su rol y sus UDN.

## Desarrollo local

```bash
npm install
cp .env.example .env   # y llena los valores
npm run dev            # la app, sin la función de usuarios
npx netlify dev        # la app + la función de usuarios
```

## Estructura

```
supabase/schema.sql              Tablas, permisos (RLS), reglas de revisión, almacenamiento
netlify/functions/admin-usuarios Alta y cambios de usuarios (usa la service_role key)
src/pages/Gastos.jsx             Tabla principal
src/pages/NuevoGasto.jsx         Captura manual / desde XML
src/pages/ImportarXml.jsx        Carga masiva de una carpeta de XML
src/pages/Catalogos.jsx          Tipos de gasto, proveedores, descripciones
src/pages/Udns.jsx, Usuarios.jsx Administración (solo super usuario)
src/lib/cfdi.js                  Lectura de CFDI 3.3 / 4.0
```
