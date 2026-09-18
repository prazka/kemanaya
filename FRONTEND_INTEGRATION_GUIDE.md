# 🌐 Panduan Integrasi Frontend: KemanaYa AI

Panduan ini ditujukan untuk rekan pengembang **Frontend (Next.js 14+)** agar dapat langsung membangun antarmuka pengguna tanpa hambatan dan terintegrasi mulus dengan backend.

---

## 1. Spesifikasi Teknis Frontend

- **Framework**: Next.js 14+ (App Router)
- **Bahasa**: TypeScript
- **Styling**: Tailwind CSS & Shadcn UI (Lucide Icons)
- **Base Backend URL**: `http://localhost:8000`
- **TypeScript Types**: Salin file `frontend_types/itinerary.ts` ke dalam folder `types/itinerary.ts` di proyek Next.js Anda.

---

## 2. Setup Proyek Next.js (Jika Belum Dibuat)

```bash
# 1. Inisialisasi Next.js
npx create-next-app@latest kemanaya-frontend --typescript --tailwind --eslint --app

# 2. Masuk ke direktori
cd kemanaya-frontend

# 3. Inisialisasi Shadcn UI
npx shadcn@latest init

# 4. Tambahkan komponen yang dibutuhkan
npx shadcn@latest add button card dialog badge tabs input select slider separator
npm install lucide-react
```

---

## 3. Client API Helper (`lib/api.ts`)

Buat file `lib/api.ts` di proyek Next.js Anda:

```typescript
import { PlanRequest, PlanResponse, DestinationPreset, BudgetAdjustRequest } from "@/types/itinerary";

const BACKEND_URL = process.env.NEXT_PUBLIC_API_URL || "http://localhost:8000";

export async function fetchHealth() {
  const res = await fetch(`${BACKEND_URL}/api/health`);
  return res.json();
}

export async function fetchDestinations(): Promise<DestinationPreset[]> {
  const res = await fetch(`${BACKEND_URL}/api/destinations`);
  if (!res.ok) throw new Error("Gagal mengambil data destinasi");
  return res.json();
}

export async function generateItinerary(payload: PlanRequest): Promise<PlanResponse> {
  const res = await fetch(`${BACKEND_URL}/api/itinerary/generate`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(payload),
  });
  if (!res.ok) {
    const errorData = await res.json().catch(() => ({}));
    throw new Error(errorData.detail || "Gagal menyusun itinerary AI");
  }
  return res.json();
}

export async function adjustBudget(payload: BudgetAdjustRequest): Promise<PlanResponse> {
  const res = await fetch(`${BACKEND_URL}/api/itinerary/adjust-budget`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(payload),
  });
  if (!res.ok) throw new Error("Gagal menghitung ulang budget");
  return res.json();
}
```

---

## 4. Struktur Halaman & Komponen yang Direkomendasikan

```text
kemanaya-frontend/
├── app/
│   ├── page.tsx                    # Landing / Input Form & Hasil Rekomendasi
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── TripPlannerForm.tsx         # Form: Destinasi, Hari, Budget, Tags
│   ├── ItineraryTimeline.tsx       # Tampilan Jadwal Hari 1, 2, 3...
│   ├── ActivityCard.tsx            # Kartu tempat wisata + rating + biaya
│   ├── VisualMediaModal.tsx        # Dialog/Modal cuplikan foto & video Reels/TikTok
│   └── BudgetBreakdownChart.tsx    # Visualisasi alokasi budget
├── types/
│   └── itinerary.ts                # (Ambil dari frontend_types/itinerary.ts)
└── lib/
    └── api.ts                      # Fetch helper
```

---

## 5. Fitur Unggulan UI untuk Demo

1. **Card Tempat Wisata Interaktif**:
   - Menampilkan thumbnail gambar HD dari `activity.media[0].thumbnail_url`.
   - Badge platform: "Instagram Reels", "TikTok", atau "Google Places".
   - Tombol "Tonton Suasana Terkini" untuk membuka dialog/modal video preview.

2. **Visualisasi Alokasi Biaya**:
   - Progress bar atau Donut chart yang menampilkan proporsi penginapan, makan, transportasi, dan tiket wisata.
   - Status badge: Hijau jika *Within Budget*, Kuning/Merah jika melebihi budget.

3. **Filter Waktu Harian**:
   - Tab "Pagi (Morning)", "Siang (Afternoon)", "Sore (Evening)", dan "Malam (Night)" untuk setiap hari.
