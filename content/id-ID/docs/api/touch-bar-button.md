## Kelas: TouchBarButton

> Buat tombol di panel sentuh untuk aplikasi macOS asli

Proses: [ Utama](../tutorial/quick-start.md#main-process)

### `TouchBarButtonbaru(pilihan)`*Eksperimental*

* `pilihan` Objek 
  * `label` String (opsional) - Teks tombol.
  * `backgroundColor` String (opsional) - Tombol warna latar dalam format hex, i.e `#ABCDEF`.
  * `ikon `[ NativeImage ](native-image.md) (opsional) - Ikon tombol.
  * `iconPosition`String (opsional) - Bisa jadi `left`,` right`atau`overlay`.
  * `klik`Fungsi (opsional) - Fungsi untuk memanggil saat tombol diklik.

### Instance Properties

Properti berikut tersedia pada contoh `TouchBarButton`:

#### `touchBarButton.label`

A `String` mewakili teks tombol saat ini. Mengubah nilai ini segera memperbarui tombolnya di bilah sentuh.

#### `touchBarButton.backgroundColor`

Kode `String` hex mewakili warna latar belakang tombol saat ini. Mengubah nilai ini segera diperbarui tombol di panel sentuh.

#### `touchBarButton.icon`

A `NativeImage` mewakili ikon tombol saat ini. Mengubah nilai ini segera memperbarui tombolnya di bilah sentuh.