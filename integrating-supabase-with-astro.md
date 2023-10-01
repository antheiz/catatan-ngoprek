# Panduan Integrasi Project Astrojs dengan Supabase

## Pendahuluan
Panduan ini akan membahas bagaimana caranya mengintegrasikan framework astro dengan supabase.

## Astro
Astro adalah sebuah framework modern untuk membangun sebuah website statis. Dengan astro kita dapat membuat sebuah website yang interactive dengan sedikit atau bahkan tanpa kode javascript. Astro juga memberikan pengalaman pengembangan bagi kita dengan pendekatan yang disebut `astro island`, dengan pendekatan ini kita dibebaskan untuk membangun suatu website dengan setiap component dari suatu halaman atau lebih yang dibangun dengan macam-macam kerangka kerja javascript seperi react, vue, atau svelte.

## Supabase
Supabase adalah sebuah product yang memiliki fungsionalitas seperti firebase, supabase dikenal sebagai alternative firebase yang berbasis opensource. Menggunakan supabase dapat membantu kita dalam membangun suatu _fullstack application_, dengan fungsionalitas seperi authentication, database, storage, realtime data akan ditangani oleh supabase. sehingga aplikasi client kita yang dibangun akan menggunakan sumber dayanya melalui API (Application Programming Interface). Supabase memiliki dokumentasi yang ramah bagi pengguna baru dan cukup mudah untuk di integrasi dengan tools atau kerangka kerja lainnya.

## Membuat Project Astro

Jalankan perintah berikut ini untuk membuat sebuah project baru untuk astro. setelah menjalankan perintah, silahkan ikuti _wizard_ yang akan mengarahkan proses pembuatan project.

`$ pnpm create astro@latest`

Selanjutnya jika kita membutuhkan _third party library_ seperti tailwind atau react, silahkan install dengan perintah berikut:

`$ pnpm astro add tailwind react`

## Inisiasi Supabase 

Pastikan anda telah memiliki akun supabase dan juga membuat suatu project baru di supabase. setelah sudah berhasil membuat project di supabase, selanjutnya silahkan navigasi ke tab SQL Editor dan paste kode sql berikut pada supabase editor untuk membuat table _(Anda bisa juga membuat secara langsung dari tab/halaman table di project supabase)_

```sql
CREATE TABLE groups (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  created_by TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);
CREATE TABLE members (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  group_id UUID REFERENCES groups(id),
  name TEXT NOT NULL,
  selected_by TEXT,
  UNIQUE (group_id, name)
);
```

Pada kode sql diatas kita akan membuat dua table, table pertama adalah groups dan table kedua adalah members dengan schema seperti terlihat pada kode sql.

## Mengatur supabase dengan Astro Project

Pertama, kita lakukan peng-installan supabase ke dependencies dengan perintah berikut:

`$ pnpm add @supabase/supabase-js`

Kemudian kita perlu membuat sebuah environment untuk mengakses supabase, maka kita membuat sebuah file bernama `.env` pada file ini akan berisi supabase url dan anon_key.

```
PUBLIC_SUPABASE_URL=<your supabase project url>
PUBLIC_SUPABASE_ANON_KEY=<your public api key>
```

> Note: pada file .env harus menggunakan kata kunci PUBLIC pada awal variabel dan perlu membuat env variabel pada file `env.d.ts`

## Generate typescript types

Sebelum melakukan generating supabase schema types, kita perlu login dengan menggunakan perintah berikut _menggunakan pnpm_:

`$ pnpm dlx supabase login`

ini akan menanyakan supabase access token, silahkan generate pada akun supabase.

Selanjutnya jika berhasil _authenticated_ silahkan jalankan perintah berikut untuk membuat types schema dari supabase table.

`$ npx supabase gen types typescript --project-id "<PROJECT_ID>" --schema public`

> _project_id_ silahkan cek pada project di supabase yang bertuliskan Reference ID.

Hasil yang akan ditampilkan setelah menjalankan perintah tersebut, silahakan copy dan paste pada sebuah file baru yang bernama `types.ts` dan berlokasi pada directory `src/` 

### Buat supabase client

Sekarang saatnya membuat supabase client pada file baru bernama `supabase.ts` yang berlokasi pada directory `src/`

```typescript
// src/supabase.ts

import { createClient } from '@supabase/supabase-js'
import type { Database } from './types'

const supabaseUrl = import.meta.env.PUBLIC_SUPABASE_URL
const supabaseAnonKey = import.meta.env.PUBLIC_SUPABASE_ANON_KEY

export const supabase = createClient<Database>(supabaseUrl, supabaseAnonKey)
```

Selanjutnya bisa mencoba koneksi dengan membuat sebuah file bernama `select.ts` yang berisi sebagai berikut:

```typescript
import {supabase} from "./supabase"

const { data, error } = await supabase
  .from('countries')
  .select()
  
console.info(data)
```

Lalu jalankan project astro dengan perintah berikut: `pnpm run dev`
Setelah project berhasil jalan, silahkan coba menambahkan suatu data pada table melalui supabase dashboard pada tab table. Setelah itu coba lakukan access url yang diberikan oleh astro melalui browser dan lihat console dari browser, jika berhasil muncul berarti proses integrasi sudah selesai.



> Tulisan v1.0.0

Ditulis pada Minggu, 11:18 PM - 01 Oktober 2023

