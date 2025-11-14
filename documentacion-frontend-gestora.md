# 📚 Documentación Frontend - Gestora

## ¿Qué es Gestora?

**Gestora** es un sistema web para gestionar la producción de quesos. Te permite controlar:

- ✅ Proveedores de materias primas
- ✅ Inventario de materias primas (leche, cuajo, sal, etc.)
- ✅ Producción de lotes de queso
- ✅ Catálogo de tipos de queso

## Tecnologías utilizadas

- **Next.js 14** - Framework de React para el frontend
- **TypeScript** - Para código más seguro
- **Tailwind CSS** - Estilos modernos y rápidos
- **Axios** - Para conectar con el backend
- **React Query** - Para manejar datos del servidor

El frontend se conecta con un backend hecho en **NestJS + PostgreSQL**.

---

## 📁 Estructura de Carpetas

```
gestora-frontend/
├── src/
│   ├── app/                          # Páginas (Next.js App Router)
│   │   ├── page.tsx                  # Dashboard principal
│   │   ├── proveedores/              # Módulo de proveedores
│   │   │   ├── page.tsx              # Lista de proveedores
│   │   │   ├── crear/page.tsx        # Crear proveedor
│   │   │   └── editar/[id]/page.tsx  # Editar proveedor
│   │   ├── materias-primas/          # Módulo de materias primas
│   │   │   ├── page.tsx              # Lista de materias primas
│   │   │   ├── crear/page.tsx
│   │   │   └── editar/[id]/page.tsx
│   │   ├── produccion/               # Módulo de producción
│   │   │   ├── page.tsx              # Lista de producciones
│   │   │   ├── crear/page.tsx
│   │   │   └── editar/[id]/page.tsx
│   │   └── quesos/                   # Catálogo de quesos
│   │       ├── page.tsx
│   │       ├── crear/page.tsx
│   │       └── editar/[id]/page.tsx
│   │
│   ├── components/                   # Componentes reutilizables
│   │   ├── Navbar.tsx                # Barra de navegación
│   │   ├── Sidebar.tsx               # Menú lateral
│   │   └── Table.tsx                 # Tabla genérica
│   │
│   ├── lib/                          # Configuración
│   │   └── api.ts                    # Configuración de Axios
│   │
│   ├── hooks/                        # Hooks personalizados
│   │   ├── useProveedores.ts
│   │   ├── useMateriasPrimas.ts
│   │   ├── useProduccion.ts
│   │   └── useQuesos.ts
│   │
│   └── types/                        # Tipos de TypeScript
│       ├── proveedor.ts
│       ├── materiaPrima.ts
│       ├── produccion.ts
│       └── queso.ts
│
├── package.json
└── README.md
```


---

## 🎯 Módulos del Sistema

### 1. Proveedores

**Qué hace:** Gestiona los proveedores que venden materias primas.

**Páginas:**
- `/proveedores` - Ver todos los proveedores
- `/proveedores/crear` - Agregar nuevo proveedor
- `/proveedores/editar/[id]` - Editar proveedor existente

**Datos del proveedor:**
```typescript
interface Proveedor {
  id: number;
  nombre: string;            // Ej: "Lácteos Don Pedro"
  contacto?: string;         // Teléfono o email
  tipoMateria: string;       // Ej: "Leche", "Cuajo"
  createdAt: string;
}
```

---

### 2. Materias Primas

**Qué hace:** Controla el inventario de ingredientes para hacer queso.

**Páginas:**
- `/materias-primas` - Ver inventario
- `/materias-primas/crear` - Agregar nueva materia prima
- `/materias-primas/editar/[id]` - Editar materia prima

**Datos de materia prima:**
```typescript
interface MateriaPrima {
  id: number;
  nombre: string;      // Ej: "Leche fresca", "Cuajo", "Sal"
  unidad: string;      // Ej: "litros", "kg", "gramos"
  stock: number;       // Cantidad disponible
}
```

**Ejemplo:**
- Leche fresca: 500 litros
- Cuajo: 2 kg
- Sal: 50 kg

---

### 3. Producción

**Qué hace:** Registra cada lote de queso producido.

**Páginas:**
- `/produccion` - Ver todas las producciones
- `/produccion/crear` - Registrar nueva producción
- `/produccion/editar/[id]` - Editar producción

**Datos de producción:**
```typescript
interface Produccion {
  id: number;
  fecha: string;              // Fecha de producción
  cantidad: number;           // Cantidad producida
  tipoQuesoId: number;        // Qué tipo de queso se hizo
  costoEstimado?: number;     // Costo total estimado
}
```

**Ejemplo:**
- Fecha: 14/11/2025
- Tipo: Queso Fresco
- Cantidad: 50 unidades
- Costo: $1,500

---

### 4. Quesos (Catálogo)

**Qué hace:** Define los tipos de queso que produces.

**Páginas:**
- `/quesos` - Ver catálogo de quesos
- `/quesos/crear` - Agregar nuevo tipo de queso
- `/quesos/editar/[id]` - Editar tipo de queso

**Datos del queso:**
```typescript
interface Queso {
  id: number;
  nombre: string;        // Ej: "Queso Fresco", "Queso Maduro"
  maduracion?: number;   // Días de maduración
  pesoProm?: number;     // Peso promedio en gramos
}
```

**Ejemplo:**
- Queso Fresco (sin maduración, 500g)
- Queso Semicurado (30 días, 800g)
- Queso Maduro (90 días, 1kg)
---

## 🔌 Conexión con el Backend

El frontend se comunica con el backend usando **Axios**.

**Archivo:** `src/lib/api.ts`

```typescript
import axios from "axios";

// Crea la conexión con el backend
export const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || "http://localhost:3000",
  headers: {
    "Content-Type": "application/json",
  },
});
```

**Variable de entorno:**
Crea un archivo `.env.local` con:
```
NEXT_PUBLIC_API_URL=http://localhost:3000
```

---

## 🪝 Hooks para Manejar Datos

Los hooks simplifican la comunicación con el backend usando **React Query**.

### Ejemplo: useProveedores

**Archivo:** `src/hooks/useProveedores.ts`

```typescript
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { api } from "@/lib/api";
import { Proveedor } from "@/types/proveedor";

export const useProveedores = () => {
  const queryClient = useQueryClient();

  // Obtener todos los proveedores
  const proveedores = useQuery({
    queryKey: ["proveedores"],
    queryFn: async () => {
      const { data } = await api.get<Proveedor[]>("/proveedores");
      return data;
    },
  });

  // Crear proveedor
  const crear = useMutation({
    mutationFn: async (nuevoProveedor: Omit<Proveedor, "id" | "createdAt">) => {
      const { data } = await api.post("/proveedores", nuevoProveedor);
      return data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["proveedores"] });
    },
  });

  // Eliminar proveedor
  const eliminar = useMutation({
    mutationFn: async (id: number) => {
      await api.delete(`/proveedores/${id}`);
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["proveedores"] });
    },
  });

  return { proveedores, crear, eliminar };
};
```

**Cómo usarlo en una página:**

```typescript
"use client";
import { useProveedores } from "@/hooks/useProveedores";

export default function ProveedoresPage() {
  const { proveedores, eliminar } = useProveedores();

  if (proveedores.isLoading) return <p>Cargando...</p>;
  if (proveedores.isError) return <p>Error al cargar proveedores</p>;

  return (
    <div>
      <h1>Proveedores</h1>
      {proveedores.data?.map((proveedor) => (
        <div key={proveedor.id}>
          <p>{proveedor.nombre}</p>
          <button onClick={() => eliminar.mutate(proveedor.id)}>
            Eliminar
          </button>
        </div>
      ))}
    </div>
  );
}
```

---

## 🎨 Componentes Reutilizables

### Tabla Genérica

**Archivo:** `src/components/Table.tsx`

```typescript
interface Column<T> {
  header: string;
  accessor: keyof T | ((row: T) => React.ReactNode);
}

interface TableProps<T> {
  data: T[];
  columns: Column<T>[];
}

export function Table<T>({ data, columns }: TableProps<T>) {
  return (
    <table className="w-full border">
      <thead>
        <tr>
          {columns.map((col, i) => (
            <th key={i} className="border p-2">{col.header}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((row, i) => (
          <tr key={i}>
            {columns.map((col, j) => (
              <td key={j} className="border p-2">
                {typeof col.accessor === "function"
                  ? col.accessor(row)
                  : String(row[col.accessor])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

---

## 🚀 Pasos para Iniciar el Proyecto

### 1. Instalar dependencias

```bash
npm install
```

### 2. Configurar variables de entorno

Crea `.env.local`:
```
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### 3. Ejecutar en desarrollo

```bash
npm run dev
```

Abre [http://localhost:3001](http://localhost:3001) en tu navegador.

### 4. Compilar para producción

```bash
npm run build
npm start
```

---

## 📝 Resumen

**Gestora** es un sistema simple y claro para:

1. **Proveedores** - ¿Quién te vende las materias primas?
2. **Materias Primas** - ¿Qué ingredientes tienes en stock?
3. **Producción** - ¿Qué lotes de queso hiciste?
4. **Quesos** - ¿Qué tipos de queso produces?

Cada módulo tiene:
- ✅ Lista (ver todos)
- ✅ Crear (agregar nuevo)
- ✅ Editar (modificar existente)
- ✅ Eliminar (borrar)

Todo conectado con el backend usando Axios y React Query para mantener los datos actualizados.