<script>
  import { onMount } from 'svelte';              // TAMBAHAN
  import { supabase } from './lib/supabase.js';  // TAMBAHAN

  console.log("Hello from App.svelte");

  let count = 0;
  function counter(){
    count ++;
    document.getElementById('counter').innerHTML = count;
  }

  function handleSubmit(e){
    e.preventDefault();
    const formData = new FormData(e.target);
    const namaKontak = formData.get('namaKontak');
    const nomorHandphone = formData.get('nomorHandphone');
    document.getElementById('nama').innerHTML = namaKontak;
    document.getElementById('kontak').innerHTML = nomorHandphone;
    simpanKontak(namaKontak, nomorHandphone);                    // TAMBAHAN (Create)
    if (namaKontak && nomorHandphone) e.target.reset();          // TAMBAHAN (kosongin form)
  }

  // ================= TAMBAHAN: CRUD ke Supabase =================
  let kontakList = [];
  let pesan = '';
  let editId = null;
  let editNama = '';
  let editNomor = '';

  // READ: ambil semua data
  async function ambilKontak() {
    const { data, error } = await supabase
      .from('kontak')
      .select('*')
      .order('id', { ascending: true });
    if (error) { pesan = 'Gagal ambil data: ' + error.message; return; }
    kontakList = data;
  }

  // CREATE: simpan data baru
  async function simpanKontak(nama, nomorHp) {
    if (!nama || !nomorHp) { pesan = 'Nama dan nomor telepon wajib diisi'; return; }
    const { error } = await supabase
      .from('kontak')
      .insert({ nama: nama, nomor_hp: nomorHp });
    if (error) { pesan = 'Gagal menyimpan: ' + error.message; return; }
    pesan = '';
    await ambilKontak();
  }

  // UPDATE: edit data (langsung di baris tabel)
  function mulaiEdit(k) {
    editId = k.id;
    editNama = k.nama;
    editNomor = k.nomor_hp;
  }
  function batalEdit() {
    editId = null;
  }
  async function simpanEdit(id) {
    if (!editNama || !editNomor) { pesan = 'Nama dan nomor telepon wajib diisi'; return; }
    const { error } = await supabase
      .from('kontak')
      .update({ nama: editNama, nomor_hp: editNomor })
      .eq('id', id);
    if (error) { pesan = 'Gagal mengubah: ' + error.message; return; }
    pesan = '';
    editId = null;
    await ambilKontak();
  }

  // DELETE: hapus data
  async function hapusKontak(id) {
    if (!confirm('Hapus kontak ini?')) return;
    const { error } = await supabase
      .from('kontak')
      .delete()
      .eq('id', id);
    if (error) { pesan = 'Gagal menghapus: ' + error.message; return; }
    pesan = '';
    await ambilKontak();
  }

  // pas halaman pertama kebuka, langsung tampilin data
  onMount(ambilKontak);
</script>

<!-- Font: Playfair Display (judul) + Quicksand (teks) -->
<svelte:head>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous">
  <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@500;600&family=Quicksand:wght@400;500;600&display=swap" rel="stylesheet">
</svelte:head>

<style>
  /*
    Palet:
    pink muda  #fbeef0 (latar)  #f6dde2  #f4b8c7
    coklat muda #fffaf6 (kartu)  #ecd5c8  #e8cdb8  #8b6450
    teks coklat tua #5b4036
  */
  :global(*) {
    box-sizing: border-box;
  }

  :global(body) {
    background-color: #fbeef0;
    color: #5b4036;
    text-align: center;
    font-family: 'Quicksand', 'Segoe UI', sans-serif;
    margin: 0;
    padding: 40px 16px;
    min-height: 100vh;
  }

  .card {
    max-width: 580px;
    margin: 0 auto;
    background-color: #fffaf6;
    border: 1px solid #ecd5c8;
    border-radius: 22px;
    padding: 32px 28px;
    box-shadow: 0 12px 32px rgba(139, 100, 80, 0.12);
  }

  /* Nama (atas) */
  #nama {
    font-family: 'Playfair Display', Georgia, serif;
    font-weight: 600;
    font-size: 2.4rem;
    color: #5b4036;
    margin: 0 0 6px;
    overflow-wrap: anywhere;
  }

  /* Kontak (bawah) */
  #kontak {
    font-family: 'Playfair Display', Georgia, serif;
    font-weight: 500;
    font-size: 1.7rem;
    color: #c4607f;
    margin: 0;
    overflow-wrap: anywhere;
  }

  #counter {
    display: table;
    margin: 20px auto;
    min-width: 64px;
    padding: 6px 20px;
    border-radius: 999px;
    background-color: #f6dde2;
    color: #55283a;
    font-size: 1.4rem;
    font-weight: 600;
  }

  .click-me-btn {
    width: 100%;
    padding: 11px;
    background-color: #e8cdb8;
    color: #4a2f27;
    border: none;
    border-radius: 12px;
    font-family: inherit;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: background-color 0.15s;
  }

  .click-me-btn:hover {
    background-color: #dfbfa6;
  }

  form {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 8px;
    margin-top: 18px;
  }

  input {
    flex: 1 1 150px;
    min-width: 0;
    background-color: #ffffff;
    color: #5b4036;
    border: 1px solid #e2c9bb;
    border-radius: 10px;
    padding: 10px 12px;
    font-family: inherit;
    font-size: 0.95rem;
  }

  input::placeholder {
    color: #b39a8c;
  }

  input:focus {
    outline: none;
    border-color: #e58fa7;
    box-shadow: 0 0 0 3px rgba(244, 184, 199, 0.5);
  }

  form button {
    background-color: #f4b8c7;
    color: #55283a;
    border: none;
    border-radius: 10px;
    padding: 10px 20px;
    font-family: inherit;
    font-size: 0.95rem;
    font-weight: 600;
    cursor: pointer;
    transition: background-color 0.15s;
  }

  form button:hover {
    background-color: #eea5b8;
  }

  button:focus-visible {
    outline: 2px solid #8b6450;
    outline-offset: 2px;
  }

  /* ===== TAMBAHAN: tabel ===== */
  .pesan {
    background-color: #f6dde2;
    color: #8a2f4a;
    border-radius: 10px;
    padding: 8px 12px;
    margin: 16px 0 0;
    font-size: 0.9rem;
  }

  .tabel-wrap {
    margin-top: 28px;
    border: 1px solid #ecd5c8;
    border-radius: 14px;
    overflow-x: auto;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    text-align: left;
  }

  th {
    background-color: #f6dde2;
    color: #5b4036;
    font-weight: 600;
    padding: 12px;
    white-space: nowrap;
  }

  td {
    padding: 10px 12px;
    border-top: 1px solid #f0e0d6;
  }

  tbody tr:nth-child(even) {
    background-color: #fdf3ee;
  }

  td.aksi {
    white-space: nowrap;
  }

  td.kosong {
    text-align: center;
    color: #9a7b6b;
    padding: 24px 12px;
  }

  td input {
    width: 100%;
    padding: 6px 8px;
    font-size: 0.9rem;
  }

  td button {
    background-color: #ecd9c9;
    color: #5b4036;
    border: none;
    border-radius: 8px;
    padding: 6px 12px;
    font-family: inherit;
    font-size: 0.85rem;
    font-weight: 600;
    cursor: pointer;
    margin-right: 4px;
    transition: background-color 0.15s;
  }

  td button:hover {
    background-color: #e3cbb7;
  }

  td button.hapus {
    background-color: #f8d0d9;
    color: #8a2f4a;
  }

  td button.hapus:hover {
    background-color: #f3bcc9;
  }

  td button.simpan {
    background-color: #8b6450;
    color: #ffffff;
  }

  td button.simpan:hover {
    background-color: #77533f;
  }
</style>

<div class="card">
  <h1 id='nama'>Nama</h1>
  <h1 id='kontak'>Kontak</h1>
  <h2 id='counter'>0</h2>
  <button class="click-me-btn" on:click={counter}>Click me</button>

  <form on:submit={handleSubmit}>
    <input type="text" name="namaKontak" placeholder="Enter your name" aria-label="Nama kontak">
    <input type="text" name="nomorHandphone" placeholder="Enter your phone number" aria-label="Nomor handphone">
    <button type="submit">Submit</button>
  </form>

  <!-- ===== TAMBAHAN: tabel data kontak ===== -->
  {#if pesan}
    <p class="pesan">{pesan}</p>
  {/if}

  <div class="tabel-wrap">
    <table>
      <thead>
        <tr>
          <th>No</th>
          <th>Nama</th>
          <th>Nomor Telepon</th>
          <th>Aksi</th>
        </tr>
      </thead>
      <tbody>
        {#each kontakList as k, i (k.id)}
          <tr>
            <td>{i + 1}</td>
            {#if editId === k.id}
              <td><input type="text" bind:value={editNama} aria-label="Ubah nama"></td>
              <td><input type="text" bind:value={editNomor} aria-label="Ubah nomor telepon"></td>
              <td class="aksi">
                <button class="simpan" on:click={() => simpanEdit(k.id)}>Simpan</button>
                <button on:click={batalEdit}>Batal</button>
              </td>
            {:else}
              <td>{k.nama}</td>
              <td>{k.nomor_hp}</td>
              <td class="aksi">
                <button on:click={() => mulaiEdit(k)}>Edit</button>
                <button class="hapus" on:click={() => hapusKontak(k.id)}>Hapus</button>
              </td>
            {/if}
          </tr>
        {:else}
          <tr>
            <td class="kosong" colspan="4">Belum ada kontak. Isi form di atas untuk menambahkan.</td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</div>
