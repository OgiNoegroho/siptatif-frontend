<template>
  <div>
    <h2>Data Mahasiswa</h2>

    <!-- Modal for Tambah Data Mahasiswa -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <span class="close-btn" @click="closeModal">&times;</span>
        <br>
        <br>
        <h3>Tambah Data Mahasiswa</h3>
        <form id="formTambahDataModal" class="form-container" @submit.prevent="tambahData">
          <div class="form-group">
            <!-- Combined field for Nama dan NIM -->
            <select v-model="selectedMahasiswa" required>
              <option value="" disabled>Pilih Mahasiswa</option>
              <option v-for="mahasiswa in mahasiswaOptions" :key="mahasiswa.NIM" :value="mahasiswa.NIM">
                {{ mahasiswa.Nama }} - {{ mahasiswa.NIM }}
              </option>
            </select>
          </div>
          <div class="form-group">
            <input type="email" v-model="inputEmail" placeholder="Email" required>
          </div>
          <div class="form-group">
            <input type="text" v-model="inputJudul" placeholder="Judul" required>
          </div>
          <div class="form-group">
            <label for="calonPembimbing1">Calon Pembimbing 1:</label>
            <select v-model="inputCalonPembimbing1" id="calonPembimbing1" @change="handlePembimbing1Change" required>
              <option value="" disabled>Pilih Pembimbing 1</option>
              <option v-for="dosen in dosenList" :key="dosen.NIP" :value="dosen.NIP">
                {{ dosen.Nama }} - {{ dosen.NIP }}
              </option>
            </select>
          </div>
          <div class="form-group">
            <label for="calonPembimbing2">Calon Pembimbing 2:</label>
            <select v-model="inputCalonPembimbing2" id="calonPembimbing2" required>
              <option value="" disabled>Pilih Pembimbing 2</option>
              <option v-for="dosen in filteredDosenList2" :key="dosen.NIP" :value="dosen.NIP">
                {{ dosen.Nama }} - {{ dosen.NIP }}
              </option>
            </select>
          </div>
          <div class="form-group">
            <select v-model="inputKategoriTA" required>
              <option value="" disabled>Pilih Kategori TA</option>
              <option value="Penelitian">Penelitian</option>
              <option value="Proyek Desain">Proyek Desain</option>
              <option value="Pengembangan Produk">Pengembangan Produk</option>
            </select>
          </div>
          <div class="form-group">
            <select v-model="inputJenisTA" required>
              <option value="" disabled>Pilih Jenis TA</option>
              <option value="Laporan">Laporan</option>
              <option value="Paper">Paper</option>
            </select>
          </div>
          <div class="form-group">
            <button class="btttn tambah-button">Tambah Data Mahasiswa</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Button to open modal -->
    <button class="btttn tambah-button" @click="openModal">
      Tambah Data Mahasiswa
    </button>

    <table class="mahasiswa-table">
      <!-- Table headers -->
      <thead>
        <tr>
          <th>No</th>
          <th>Tanggal</th>
          <th>Nama</th>
          <th>NIM</th>
          <th>Email</th>
          <th>Judul</th>
          <th>Status</th>
          <th>Aksi</th>
        </tr>
      </thead>
      <!-- Table body -->
      <tbody>
        <!-- Loop through mahasiswaList array -->
        <tr v-for="(mahasiswa, index) in mahasiswaList" :key="mahasiswa.nim">
          <!-- Display data for each mahasiswa -->
          <td>{{ index + 1 }}</td>
          <td>{{ mahasiswa.tanggal }}</td>
          <td>{{ mahasiswa.nama }}</td>
          <td>{{ mahasiswa.nim }}</td>
          <td>{{ mahasiswa.email }}</td>
          <td>{{ mahasiswa.judul }}</td>
          <td>{{ mahasiswa.status }}</td>
          <!-- Action buttons -->
          <td>
            <div class="aksi-buttons">
              <!-- Button to show details -->
              <button class="btttn tombol-detail" @click="showDetail(mahasiswa)">Detail</button>
              <!-- Button to delete -->
              <button class="btttn tombol-delete" @click="confirmDelete(mahasiswa)">Hapus</button>
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Card Layout for Mobile View -->
    <div class="card-container" v-for="(mahasiswa, index) in mahasiswaList" :key="mahasiswa.nim">
      <div class="card">
        <div class="card-content">
          <p><strong>No:</strong> {{ index + 1 }}</p>
          <p><strong>Tanggal:</strong> {{ mahasiswa.tanggal }}</p>
          <p><strong>Nama:</strong> {{ mahasiswa.nama }}</p>
          <p><strong>NIM:</strong> {{ mahasiswa.nim }}</p>
          <p><strong>Email:</strong> {{ mahasiswa.email }}</p>
          <p><strong>Judul:</strong> {{ mahasiswa.judul }}</p>
          <p><strong>Status:</strong> {{ mahasiswa.status }}</p>
          <div class="aksi-buttons">
            <button class="btttn tombol-detail" @click="showDetail(mahasiswa)">Detail</button>
            <button class="btttn tombol-delete" @click="confirmDelete(mahasiswa)">Hapus</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { useToast } from 'vue-toastification';

export default {
  data() {
    return {
      mahasiswaList: [],
      mahasiswaOptions: [],
      showModal: false,
      selectedMahasiswa: '',
      inputEmail: '',
      inputJudul: '',
      inputCalonPembimbing1: '',
      inputCalonPembimbing2: '',
      inputKategoriTA: '',
      inputJenisTA: '',
      dosenList: [],
      filteredDosenList2: [] // To store the filtered list for Pembimbing 2
    };
  },
  methods: {
    async fetchData() {
      try {
        const response = await axios.get('https://express-mysql-virid.vercel.app/api/pendaftaran', {
          headers: {
            'Authorization': `Bearer ${localStorage.getItem('token')}`
          }
        });
        this.mahasiswaList = response.data;
        // Format the date
        this.mahasiswaList.forEach(mahasiswa => {
          mahasiswa.tanggal = new Date(mahasiswa.tanggal).toISOString().split('T')[0];
        });
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    },
    async fetchMahasiswaOptions() {
      try {
        const response = await axios.get('https://express-mysql-virid.vercel.app/api/mahasiswa', {
          headers: {
            'Authorization': `Bearer ${localStorage.getItem('token')}`
          }
        });
        this.mahasiswaOptions = response.data;
      } catch (error) {
        console.error('Error fetching mahasiswa options:', error);
      }
    },
    async fetchDosen() {
      try {
        const response = await axios.get('https://express-mysql-virid.vercel.app/api/dosen', {
          headers: {
            'Authorization': `Bearer ${localStorage.getItem('token')}`
          }
        });
        this.dosenList = response.data || [];
      } catch (error) {
        console.error('Error fetching dosen data:', error);
      }
    },
    async fetchDosenWithoutNIP(nip) {
      try {
        const response = await axios.get(`https://express-mysql-virid.vercel.app/api/dosen/without/${nip}`, {
          headers: {
            'Authorization': `Bearer ${localStorage.getItem('token')}`
          }
        });
        this.filteredDosenList2 = response.data;
      } catch (error) {
        console.error('Error fetching filtered dosen list:', error);
      }
    },
    showDetail(mahasiswa) {
      this.$router.push({ name: 'FileMahasiswa', params: { nim: mahasiswa.nim } });
    },
    async confirmDelete(mahasiswa) {
      const toast = useToast();
      if (confirm("Apakah Anda yakin ingin menghapus data mahasiswa ini?")) {
        try {
          await axios.delete(`https://express-mysql-virid.vercel.app/api/pendaftaran/${mahasiswa.nim}`, {
            headers: {
              'Authorization': `Bearer ${localStorage.getItem('token')}`
            }
          });
          const index = this.mahasiswaList.findIndex(item => item.nim === mahasiswa.nim);
          if (index !== -1) {
            this.mahasiswaList.splice(index, 1);
          }
          toast.success('Data mahasiswa berhasil dihapus');
        } catch (error) {
          toast.error('Terjadi kesalahan saat menghapus data mahasiswa');
          console.error('Error deleting data:', error);
        }
      } else {
        console.log("Penghapusan data dibatalkan");
      }
    },
    openModal() {
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
      this.resetForm();
    },
    resetForm() {
      this.selectedMahasiswa = '';
      this.inputEmail = '';
      this.inputJudul = '';
      this.inputCalonPembimbing1 = '';
      this.inputCalonPembimbing2 = '';
      this.inputKategoriTA = '';
      this.inputJenisTA = '';
    },
    async tambahData() {
      const toast = useToast();
      try {
        const NIM = this.selectedMahasiswa;
        const data = {
          NIM,
          Judul_TA: this.inputJudul,
          KategoriTA: this.inputKategoriTA,
          JenisTA: this.inputJenisTA,
          nip_pembimbing1: this.inputCalonPembimbing1,
          nip_pembimbing2: this.inputCalonPembimbing2,
          nip_penguji1: null,
          nip_penguji2: null,
          status: 'menunggu'
        };

        await axios.post('https://express-mysql-virid.vercel.app/api/pendaftaran', data, {
          headers: {
            'Authorization': `Bearer ${localStorage.getItem('token')}`
          }
        });

        toast.success('Data mahasiswa berhasil ditambahkan');
        this.closeModal();
        this.fetchData();
      } catch (error) {
        toast.error('Terjadi kesalahan saat menambahkan data mahasiswa');
        console.error('Error adding data:', error);
      }
    },
    handlePembimbing1Change() {
      this.fetchDosenWithoutNIP(this.inputCalonPembimbing1);
    }
  },
  mounted() {
    this.fetchData();
    this.fetchDosen();
    this.fetchMahasiswaOptions();
  }
};
</script>


<style scoped>
/* Global styles */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Poppins', sans-serif;
}

h2 {
  margin-top: 20px;
  margin-left: 20px;
}

h3 {
  text-align: center;
  padding-left: 10%;
  flex-grow: 1;
}

.mahasiswa-table {
  width: 96.5%;
  max-width: 2000px;
  border-collapse: collapse;
  margin: 20px auto; /* Center the table */
}

.mahasiswa-table th,
.mahasiswa-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.mahasiswa-table th {
  background-color: #f2f2f2;
}

.mahasiswa-table tr:nth-child(even) {
  background-color: #f2f2f2;
}

.mahasiswa-table tr:hover {
  background-color: #ddd;
}

table tbody tr td:last-child {
  display: flex;
  justify-content: center;
  align-items: center;
}

.aksi-buttons {
  display: flex;
}

.btttn {
  padding: 8px 12px;
  cursor: pointer;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  text-align: center;
  text-decoration: none;
}

.tombol-detail {
  background-color: #2196F3;
  color: white;
  margin-right: 2px;
}

.tombol-delete {
  background-color: #f44336;
  color: white;
}

.tambah-button {
  background-color: #4CAF50;
  color: white;
  margin-left: 20px;
  margin-top: 20px;
}

.modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  z-index: 1002;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  background-color: #fff;
  padding: 15px;
  border: 1px solid #888;
  width: 80%;
  max-width: 400px;
  border-radius: 8px;
}

.close-btn {
  color: #030303;
  float: right;
  margin-bottom: 20px;
  margin-left: 20px;
  font-size: 28px;
  font-weight: bold;
}

.close-btn:hover,
.close-btn:focus {
  color: rgb(241, 4, 4);
  text-decoration: none;
  cursor: pointer;
}

.form-container {
  display: flex;
  flex-direction: column;
}

.form-group {
  margin-bottom: 10px;
}

.form-group input,
.form-group select {
  width: 100%;
  padding: 8px;
  margin: 5px 0;
  display: inline-block;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.form-group label {
  margin-bottom: 5px;
}

.form-group button {
  width: 88%;
  background-color: #4CAF50;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.form-group button:hover {
  background-color: #45a049;
}

/* Card styles */
.card-container {
  display: none;
}

.card {
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 8px;
  margin: 19px;
  padding: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.card-content p {
  margin: 8px 0;
}

.card .aksi-buttons {
  display: flex;
  justify-content: space-around;
  margin-top: 10px;
}

@media (max-width: 769px) {
  .mahasiswa-table {
    display: none;
  }

  .card-container {
    display: block;
  }

  .card .aksi-buttons {
    display: flex;
    justify-content: space-between; /* Menyesuaikan agar tombol berada di samping kanan kiri */
    margin-top: 10px;
  }

  .card .aksi-buttons button {
    flex: 1; /* Memastikan tombol menggunakan ruang yang sama */
  }
}
</style>