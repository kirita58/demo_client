<!-- File: src/pages/lich_lam_viec/LichCaLam.vue -->
<template>
  <div class="lich-page">
    <div class="header-section">
      <h2 class="page-title">Quản Lý Ca Làm Việc</h2>
    </div>
    <!-- toast status messages (same style used in taikhoan_nhanvien) -->
    <div v-if="pageToast.show" class="ss-page-toast" :class="pageToast.type">
      <span class="material-icons-outlined ss-page-toast-ic">
        {{ pageToast.type === "success" ? "check_circle" : pageToast.type === "error" ? "error" : "info" }}
      </span>
      <div class="ss-page-toast-msg">{{ pageToast.msg }}</div>
      <button class="ss-page-toast-x" type="button" @click="hidePageToast">×</button>
    </div>

    <div v-if="confirmTrangThai.show" class="ss-confirm-toast">
      <span class="material-icons-outlined ss-confirm-ic">help_outline</span>
      <div class="ss-confirm-msg">{{ confirmTrangThai.msg }}</div>

      <div class="ss-confirm-actions">
        <button class="ss-confirm-btn ss-confirm-cancel" type="button" @click="cancelConfirmTrangThai"
          :disabled="confirmTrangThai.loading">
          Hủy
        </button>
        <button class="ss-confirm-btn ss-confirm-ok" type="button" @click="okConfirmTrangThai"
          :disabled="confirmTrangThai.loading">
          {{ confirmTrangThai.loading ? "Đang cập nhật..." : "Xác nhận" }}
        </button>
      </div>
    </div>

    <div class="panel">
      <div class="toolbar">
        <div class="toolbar-left">
          <div class="search-wrapper">
            <i class="fa-solid fa-magnifying-glass search-icon"></i>
            <input v-model="filters.keyword" type="text" placeholder="Nhập tên ca, mã ca..." class="search-input" />
          </div>
        </div>

        <div class="toolbar-right">
          <button class="btn btn-reset" @click="resetFilter" type="button">
            <span class="material-icons-outlined btn-mi">restart_alt</span>
            Đặt lại bộ lọc
          </button>
        </div>
      </div>

      <div class="filters-bar">
        <div class="filter-group">
          <label class="ss-label">Giờ bắt đầu:</label>
          <input type="time" v-model="filters.gioBatDau" class="filter-pill" />
        </div>

        <div class="filter-group">
          <label class="ss-label">Giờ kết thúc:</label>
          <input type="time" v-model="filters.gioKetThuc" class="filter-pill" />
        </div>

        <div class="filter-group">
          <label class="ss-label">Trạng thái:</label>
          <select v-model="filters.trangThai" class="filter-pill">
            <option value="all">Tất cả</option>
            <option value="active">Hoạt động</option>
            <option value="inactive">Ngưng</option>
          </select>
        </div>
      </div>
    </div>

    <div class="card-box mt-20">
      <div class="table-header-row">
        <h3>Danh sách Ca làm việc</h3>

        <div class="action-buttons">
          <button v-if="hasPermission" class="btn-circle btn-success" @click="openModal(null)" title="Thêm mới">
            <span><i class="fa-solid fa-plus"></i></span>
          </button>
          <button class="btn-circle btn-light" @click="loadData" title="Làm mới">
            <span><i class="fa-solid fa-rotate-right"></i></span>
          </button>
        </div>
      </div>

      <div class="table-responsive mt-10">
        <table>
          <thead>
            <tr>
              <th style="width: 60px" class="text-center">STT</th>
              <th>Thông tin Ca</th>
              <th class="text-center">Giờ bắt đầu</th>
              <th class="text-center">Giờ kết thúc</th>
              <th class="text-center">Trạng thái</th>
              <th class="text-center" style="width: 120px">Thao tác</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(ca, index) in danhSachCaLam" :key="ca.id">
              <td class="text-center">{{ index + 1 }}</td>
              <td>
                <div class="shift-info">
                  <div class="shift-icon">
                    <span><i class="fa-regular fa-clock"></i></span>
                  </div>
                  <div class="shift-details">
                    <span class="shift-name">{{ ca.tenCa }}</span>
                    <span class="shift-code">{{ ca.maCa }}</span>
                  </div>
                </div>
              </td>
              <td class="text-center">
                <span class="time-badge start">{{ ca.gioBatDau }}</span>
              </td>
              <td class="text-center">
                <span class="time-badge end">{{ ca.gioKetThuc }}</span>
              </td>
              <td class="text-center">
                <label class="switch">
                  <input type="checkbox" :checked="ca.trangThai" :disabled="!hasPermission"
                    @click.stop.prevent="toggleTrangThai(ca)">
                  <span class="slider round"></span>
                </label>
              </td>
              <td class="text-center action-col">
                <button class="ss-icon-btn-view" type="button" @click="openModal(ca)">
                  <span class="material-icons-outlined">visibility</span>
                </button>
              </td>
            </tr>

            <tr v-if="danhSachCaLam.length === 0">
              <td colspan="6" class="text-center py-4">Không có dữ liệu ca làm việc</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div v-if="showModal" class="modal-overlay">
      <div class="modal-content">
        <div class="modal-header">
          <h3>{{ isEditing ? 'Cập nhật Ca làm việc' : 'Thêm mới Ca làm việc' }}</h3>
          <button @click="closeModal" class="btn-close" type="button" title="Đóng">×</button>
        </div>

        <div class="modal-body">
          <div class="form-group mb-15">
            <label>Tên ca <span class="text-danger">*</span></label>
            <input type="text" class="form-control" v-model="form.tenCa" placeholder="VD: Ca Sáng, Ca Chiều..." />
          </div>

          <div class="row-flex mb-15">
            <div class="form-group flex-1 mr-10">
              <label>Giờ bắt đầu</label>
              <input type="time" lang="en-GB" class="form-control" v-model="form.gioBatDau" />
            </div>
            <div class="form-group flex-1">
              <label>Giờ kết thúc</label>
              <input type="time" lang="en-GB" class="form-control" v-model="form.gioKetThuc" />
            </div>
          </div>

          <div class="form-group mb-15">
            <label>Mô tả</label>
            <textarea class="form-control textarea" v-model="form.moTa" rows="3"
              placeholder="Ghi chú thêm về ca làm việc..."></textarea>
          </div>
        </div>

        <div class="modal-footer">
          <button class="btn-cancel" type="button" @click="closeModal">Hủy bỏ</button>
          <button v-if="hasPermission" class="btn-save" type="button" @click="handleSubmit">
            {{ isEditing ? 'Lưu' : 'Thêm mới' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { createCaLam, getAllCaLam, updateCaLam } from '@/services/lich_lam_viec/ca_lamService';
import { computed, ref, reactive, onMounted, watch } from 'vue';

const getUser = () => {
  const raw = localStorage.getItem("user") || sessionStorage.getItem("user") ||
    localStorage.getItem("nguoiDung") || sessionStorage.getItem("nguoiDung");
  if (!raw) return null;
  try { return JSON.parse(raw); } catch { return null; }
};

const hasPermission = computed(() => {
  const u = getUser();
  const role = u?.role || u?.quyen || u?.vaiTro || u?.tenVaiTro;

  if (role === "NHAN_VIEN") {
    return false;
  }


  return true;
});

const danhSachCaLam = ref([]);
const caLamOrigin = ref([]);

const showModal = ref(false);
const isEditing = ref(false);
const currentId = ref(null);

const pageToast = reactive({ show: false, type: 'info', msg: '' });
let pageToastTimer = null;

const showPageToast = (type, msg) => {
  pageToast.show = true;
  pageToast.type = type || 'info';
  pageToast.msg = msg || '';

  if (pageToastTimer) clearTimeout(pageToastTimer);
  pageToastTimer = setTimeout(() => {
    pageToast.show = false;
  }, 2600);
};

const hidePageToast = () => {
  pageToast.show = false;
};

const confirmTrangThai = reactive({
  show: false,
  loading: false,
  msg: '',
  item: null,
  newValue: false,
});

const openConfirmTrangThai = (ca, newValue) => {
  const ten = (ca?.tenCa ?? '').toString().trim() || 'ca';
  const nextText = newValue ? 'Hoạt động' : 'Ngưng hoạt động';

  confirmTrangThai.show = true;
  confirmTrangThai.loading = false;
  confirmTrangThai.item = ca;
  confirmTrangThai.newValue = !!newValue;
  confirmTrangThai.msg = `Bạn có muốn chuyển trạng thái của "${ten}" sang "${nextText}" không?`;
};

const cancelConfirmTrangThai = () => {
  if (confirmTrangThai.loading) return;
  confirmTrangThai.show = false;
  confirmTrangThai.loading = false;
  confirmTrangThai.msg = '';
  confirmTrangThai.item = null;
  confirmTrangThai.newValue = false;
};

const capNhatTrangThai = async (ca, newValue) => {
  const id = ca?.id;
  if (id == null) return;

  const nextValue = !!newValue;

  try {
    const payload = {
      ...ca,
      gioBatDau: ca.gioBatDau && ca.gioBatDau.length === 5 ? `${ca.gioBatDau}:00` : ca.gioBatDau,
      gioKetThuc: ca.gioKetThuc && ca.gioKetThuc.length === 5 ? `${ca.gioKetThuc}:00` : ca.gioKetThuc,
      trangThai: nextValue,
    };

    await updateCaLam(id, payload);

    ca.trangThai = nextValue;

    showPageToast('success', 'Đã cập nhật trạng thái');
  } catch (error) {
    console.error('Lỗi khi cập nhật trạng thái:', error);
    showPageToast('error', 'Không thể cập nhật trạng thái. Vui lòng thử lại.');
  }
};

const okConfirmTrangThai = async () => {
  const ca = confirmTrangThai.item;
  if (!ca || ca.id == null) {
    cancelConfirmTrangThai();
    return;
  }
  if (confirmTrangThai.loading) return;

  confirmTrangThai.loading = true;
  try {
    await capNhatTrangThai(ca, confirmTrangThai.newValue);
  } finally {
    confirmTrangThai.loading = false;
    cancelConfirmTrangThai();
  }
};

// State bộ lọc
const filters = reactive({
  keyword: '',
  gioBatDau: '',
  gioKetThuc: '',
  trangThai: 'all'
});

const form = reactive({
  tenCa: '',
  gioBatDau: '',
  gioKetThuc: '',
  moTa: '',
  trangThai: true
});

const handleFilter = () => {
  let filtered = [...caLamOrigin.value];

  // 1. Lọc theo từ khóa (tên ca, mã ca)
  if (filters.keyword.trim()) {
    const kw = filters.keyword.trim().toLowerCase();
    filtered = filtered.filter(ca =>
      (ca.tenCa && ca.tenCa.toLowerCase().includes(kw)) ||
      (ca.maCa && ca.maCa.toLowerCase().includes(kw))
    );
  }

  // 2. Lọc theo giờ bắt đầu
  if (filters.gioBatDau) {
    filtered = filtered.filter(ca => {
      if (!ca.gioBatDau) return false;
      return ca.gioBatDau.startsWith(filters.gioBatDau);
    });
  }

  // 3. Lọc theo giờ kết thúc
  if (filters.gioKetThuc) {
    filtered = filtered.filter(ca => {
      if (!ca.gioKetThuc) return false;
      return ca.gioKetThuc.startsWith(filters.gioKetThuc);
    });
  }

  // 4. Lọc theo trạng thái
  if (filters.trangThai !== 'all') {
    const isActive = filters.trangThai === 'active';
    filtered = filtered.filter(ca => !!ca.trangThai === isActive);
  }

  // Gán lại kết quả đã lọc cho bảng hiển thị
  danhSachCaLam.value = filtered;
};

const resetFilter = () => {
  filters.keyword = '';
  filters.gioBatDau = '';
  filters.gioKetThuc = '';
  filters.trangThai = 'all';
};

const loadData = async () => {
  try {
    const res = await getAllCaLam();
    caLamOrigin.value = Array.isArray(res) ? res : res.content || [];
    handleFilter();
  } catch (e) {
    console.error(e);
  }
};

const openModal = (ca) => {
  if (ca) {
    isEditing.value = true;
    currentId.value = ca.id;

    form.tenCa = ca.tenCa || '';

    form.gioBatDau = ca.gioBatDau ? String(ca.gioBatDau).substring(0, 5) : '';
    form.gioKetThuc = ca.gioKetThuc ? String(ca.gioKetThuc).substring(0, 5) : '';

    form.moTa = ca.moTa || '';
    form.trangThai = ca.trangThai !== undefined ? ca.trangThai : true;
  } else {
    isEditing.value = false;
    currentId.value = null;
    form.tenCa = '';
    form.gioBatDau = '';
    form.gioKetThuc = '';
    form.moTa = '';
    form.trangThai = true;
  }
  showModal.value = true;
};

const closeModal = () => {
  showModal.value = false;
};

const handleSubmit = async () => {
  const tenCaTrimmed = form.tenCa.trim();

  const isDuplicate = caLamOrigin.value.some(ca => 
    ca.tenCa && 
    ca.tenCa.toLowerCase() === tenCaTrimmed.toLowerCase() && 
    ca.id !== currentId.value
  );

  if (isDuplicate) {
    showPageToast('error', 'Tên ca này đã tồn tại!');
    return;
  }

  if (tenCaTrimmed.length === 0) {
    showPageToast('error', 'Tên ca không được để trống!');
    return;
  }

  if (tenCaTrimmed.length > 50) {
    showPageToast('error', 'Tên ca không được vượt quá 50 kí tự!');
    return;
  }

  const kitudacbiet = /[~`!@#$%^&*()+=\-[\]\\';,/{}|\\":<>?]/g;

  if (kitudacbiet.test(tenCaTrimmed)) {
    showPageToast('error', 'Tên ca không được chứa ký tự đặc biệt!');
    return;
  }

  if (!form.gioBatDau || !form.gioKetThuc) {
    showPageToast('error', 'Vui lòng chọn đầy đủ giờ bắt đầu và giờ kết thúc!');
    return;
  }

  if (form.gioBatDau === form.gioKetThuc) {
    showPageToast('error', 'Giờ bắt đầu và giờ kết thúc không được trùng nhau!');
    return;
  }

  if (/^\d+$/.test(tenCaTrimmed)) {
    showPageToast('error', 'Tên ca không được chỉ chứa toàn số!');
    return;
  }
  
  const payload = {
    tenCa: tenCaTrimmed,
    gioBatDau: form.gioBatDau ? `${form.gioBatDau}:00` : null,
    gioKetThuc: form.gioKetThuc ? `${form.gioKetThuc}:00` : null,
    moTa: form.moTa.trim(),
    trangThai: form.trangThai
  };

  try {
    if (isEditing.value) {
      await updateCaLam(currentId.value, payload);
      showPageToast('success', 'Cập nhật thành công!');
    } else {
      await createCaLam(payload);
      showPageToast('success', 'Thêm mới thành công!');
    }
    closeModal();
    loadData();
  } catch (error) {
    console.error('Lỗi khi lưu ca làm việc:', error);

    let errorMsg = 'Có lỗi xảy ra, vui lòng thử lại.';

    if (error.response && error.response.data && error.response.data.message) {
      errorMsg = error.response.data.message;
    } else if (error.message) {
      try {
        const parsed = JSON.parse(error.message.replace('Error: ', ''));
        if (parsed.message) errorMsg = parsed.message;
      } catch {
        errorMsg = error.message;
      }
    }

    showPageToast('error', errorMsg);
  }
};


const toggleTrangThai = (ca) => {
  if (!hasPermission.value) return;
  const newValue = !ca.trangThai;
  openConfirmTrangThai(ca, newValue);
};

watch(
  filters,
  () => {
    handleFilter();
  },
  { deep: true }
);

onMounted(() => {
  loadData();
});
</script>

<style scoped>
.lich-page {
  font-family: var(--admin-font, sans-serif);
  padding: 20px;
  background-color: #f8f9fa;
  min-height: 100vh;
}

.page-title {
  font-size: 22px;
  font-weight: 700;
  margin-bottom: 30px;
  margin-top: 10px;
  color: rgba(17, 24, 39, 0.92);
}

.card-box {
  background: white;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.03);
  padding: 20px;
  border: 1px solid #f3f4f6;
}

.mt-20 {
  margin-top: 20px;
}

.mt-10 {
  margin-top: 10px;
}

.text-center {
  text-align: center;
}

.py-4 {
  padding-top: 1.5rem;
  padding-bottom: 1.5rem;
}

/* Filter Header */
.filter-header,
.table-header-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #f3f4f6;
  padding-bottom: 15px;
  margin-bottom: 15px;
}

.filter-header h3,
.table-header-row h3 {
  font-size: 16px;
  font-weight: 600;
  color: #111827;
  margin: 0;
}

.btn-icon-bg {
  background: #f3f4f6;
  border: none;
  width: 32px;
  height: 32px;
  border-radius: 6px;
  color: #6b7280;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.btn-icon-bg:hover {
  background: #e5e7eb;
  color: #111827;
}

.filter-body {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.filter-col {
  flex: 1;
  min-width: 200px;
}

.form-group label {
  display: block;
  font-size: 13px;
  font-weight: 500;
  color: #4b5563;
  margin-bottom: 8px;
}

.form-control {
  width: 100%;
  height: 40px;
  padding: 8px 12px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  font-size: 14px;
  color: #1f2937;
  transition: border-color 0.2s;
}

.form-control:focus {
  outline: none;
  border-color: #ff4d4f;
}

/* Custom Inputs */
.search-input-wrapper {
  position: relative;
}

.search-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: #9ca3af;
}

.pl-35 {
  padding-left: 35px;
}

/* Radio Buttons Custom */
.radio-group {
  display: flex;
  gap: 15px;
  height: 40px;
  align-items: center;
}

.radio-container {
  display: flex;
  align-items: center;
  position: relative;
  padding-left: 24px;
  cursor: pointer;
  font-size: 14px;
  user-select: none;
  color: #374151;
}

.radio-container input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
}

.checkmark {
  position: absolute;
  top: 50%;
  left: 0;
  transform: translateY(-50%);
  height: 16px;
  width: 16px;
  background-color: #fff;
  border: 1px solid #d1d5db;
  border-radius: 50%;
}

.radio-container input:checked ~ .checkmark {
  border-color: #10b981;
}

.checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

.radio-container input:checked ~ .checkmark:after {
  display: block;
}

.radio-container .checkmark:after {
  top: 3px;
  left: 3px;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #10b981;
}

.action-buttons {
  display: flex;
  gap: 10px;
}

.btn-circle {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: white;
  font-size: 14px;
  transition: all 0.2s;
}

.btn-success {
  background-color: #ff4d4f;
}

.btn-success:hover {
  background-color: #ff4d4f;
  box-shadow: 0 4px 10px rgba(16, 185, 129, 0.3);
}

.btn-light {
  background-color: #f3f4f6;
  color: #4b5563;
}

.btn-light:hover {
  background-color: #e5e7eb;
}

/* Table Styling */
table {
  width: 100%;
  border-collapse: collapse;
}

th {
  font-weight: 600;
  color: #374151;
  font-size: 13px;
  padding: 12px;
  border-bottom: 1px solid #e5e7eb;
  background-color: #f9fafb;
}

td {
  padding: 14px 12px;
  border-bottom: 1px solid #f3f4f6;
  font-size: 14px;
  vertical-align: middle;
}

tbody tr:hover {
  background-color: #f9fafb;
}

/* Shift Info Cell */
.shift-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

.shift-icon {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: #e0f2fe;
  color: #0284c7;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
}

.shift-details {
  display: flex;
  flex-direction: column;
}

.shift-name {
  font-weight: 600;
  color: #111827;
  margin-bottom: 2px;
}

.shift-code {
  font-size: 12px;
  color: #6b7280;
}

/* Time Badges */
.time-badge {
  padding: 4px 12px;
  border-radius: 20px;
  font-weight: 600;
  font-size: 13px;
}

.time-badge.start {
  background-color: #d1fae5;
  color: #059669;
}

.time-badge.end {
  background-color: #fee2e2;
  color: #dc2626;
}

/* Toggle Switch */
.switch {
  position: relative;
  display: inline-block;
  width: 44px;
  height: 24px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #cbd5e1;
  transition: .4s;
}

.slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 3px;
  bottom: 3px;
  background-color: white;
  transition: .4s;
}

input:checked + .slider {
  background-color: #ff4d4f;
}

input:checked + .slider:before {
  transform: translateX(20px);
}

.slider.round {
  border-radius: 34px;
}

.slider.round:before {
  border-radius: 50%;
}

/* Action Icons in Table */
.action-icon-btn {
  background: none;
  border: none;
  font-size: 16px;
  margin: 0 4px;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
}

.action-icon-btn:hover {
  background: #f3f4f6;
}

.text-warning {
  color: #d97706;
}

.text-danger {
  color: #dc2626;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(17, 24, 39, 0.4);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  backdrop-filter: blur(2px);
}

.modal-content {
  background: white;
  width: 100%;
  max-width: 500px;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  animation: slideDown 0.3s ease;
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  border-bottom: 1px solid #f3f4f6;
}

.modal-header h3 {
  margin: 0;
  font-size: 16px;
  font-weight: 600;
  color: #111827;
}

.btn-close {
  background: none;
  border: none;
  font-size: 24px;
  color: #9ca3af;
  cursor: pointer;
  line-height: 1;
}

.btn-close:hover {
  color: #ef4444;
}

.modal-body {
  padding: 20px;
}

.mb-15 {
  margin-bottom: 15px;
}

.text-danger {
  color: #ef4444;
}

.row-flex {
  display: flex;
  align-items: flex-end;
}

.flex-1 {
  flex: 1;
}

.mr-10 {
  margin-right: 10px;
}

.textarea {
  resize: vertical;
  min-height: 80px;
}

.text-status {
  font-size: 14px;
  font-weight: 500;
  color: #374151;
}

.modal-footer {
  padding: 16px 20px;
  border-top: 1px solid #f3f4f6;
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  background: #f9fafb;
}

.btn-cancel {
  padding: 8px 16px;
  background: white;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  color: #374151;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-cancel:hover {
  background: #f3f4f6;
}

.btn-save {
  padding: 8px 16px;
  background: linear-gradient(90deg, #ff4d4f 0%, #111827 100%);
  box-shadow: 0 10px 18px rgba(255, 77, 79, 0.16);
  border: none;
  border-radius: 6px;
  color: white;
  font-weight: 500;
  cursor: pointer;
}

.btn-save:hover {
  background: #ff4d4f;
}

.ss-icon-btn-view,
.ss-icon-btn-danger {
  width: 34px;
  height: 34px;
  border-radius: 8px;
  border: 1px solid #e5e7eb;
  background: #fff;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  transition: all 0.15s ease;
}

.ss-icon-btn-view:hover {
  background: #f3f4f6;
}

.ss-icon-btn-danger:hover {
  background: #fef2f2;
  border-color: #fecaca;
}

/* =======================
   ✅ TOAST (TRANG)
   ======================= */
.ss-page-toast {
  position: fixed;
  top: 14px;
  right: 14px;
  z-index: 2500;
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 280px;
  max-width: 460px;
  padding: 10px 12px;
  border-radius: 12px;
  background: #fff;
  border: 1px solid rgba(17, 24, 39, 0.12);
  box-shadow: 0 18px 45px rgba(17, 24, 39, 0.14);
}

.ss-page-toast.success {
  border-color: rgba(34, 197, 94, 0.25);
}

.ss-page-toast.error {
  border-color: rgba(239, 68, 68, 0.25);
}

.ss-page-toast.info {
  border-color: rgba(59, 130, 246, 0.25);
}

.ss-page-toast-ic {
  font-size: 18px;
  color: rgba(17, 24, 39, 0.55);
}

.ss-page-toast.success .ss-page-toast-ic {
  color: rgba(34, 197, 94, 0.95);
}

.ss-page-toast.error .ss-page-toast-ic {
  color: rgba(239, 68, 68, 0.95);
}

.ss-page-toast.info .ss-page-toast-ic {
  color: rgba(59, 130, 246, 0.95);
}

.ss-page-toast-msg {
  color: rgba(17, 24, 39, 0.86);
  font-size: 13px;
  line-height: 1.35;
  flex: 1;
}

.ss-page-toast-x {
  border: none;
  background: transparent;
  cursor: pointer;
  font-size: 18px;
  line-height: 1;
  color: rgba(17, 24, 39, 0.45);
}

.ss-page-toast-x:hover {
  color: rgba(17, 24, 39, 0.7);
}

/* =======================
   ✅ TOAST XÁC NHẬN
   ======================= */
.ss-confirm-toast {
  position: fixed;
  top: 64px;
  right: 14px;
  z-index: 2501;
  display: flex;
  align-items: flex-start;
  gap: 10px;
  width: min(520px, calc(100vw - 28px));
  padding: 12px;
  border-radius: 14px;
  background: #fff;
  border: 1px solid rgba(255, 77, 79, 0.25);
  box-shadow: 0 18px 55px rgba(17, 24, 39, 0.18);
}

.ss-confirm-ic {
  font-size: 20px;
  color: rgba(255, 77, 79, 0.95);
  margin-top: 1px;
}

.ss-confirm-msg {
  flex: 1;
  color: rgba(17, 24, 39, 0.86);
  font-size: 13.5px;
  line-height: 1.35;
  padding-right: 8px;
}

.ss-confirm-actions {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  margin-top: 2px;
}

.ss-confirm-btn {
  height: 32px;
  padding: 0 12px;
  border-radius: 10px;
  border: 1px solid transparent;
  cursor: pointer;
  font-size: 12.5px;
  font-weight: 700;
  transition: 0.15s ease;
}

.ss-confirm-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.ss-confirm-cancel {
  background: #f3f4f6;
  border-color: rgba(17, 24, 39, 0.12);
  color: rgba(17, 24, 39, 0.82);
}

.ss-confirm-cancel:hover {
  background: #eef0f3;
}

.ss-confirm-ok {
  background: #ff4d4f;
  border-color: rgba(255, 77, 79, 0.35);
  color: #fff;
  box-shadow: 0 10px 18px rgba(255, 77, 79, 0.16);
}

.ss-confirm-ok:hover {
  filter: brightness(0.98);
}

/* =======================
   ✅ BỘ LỌC TÌM KIẾM (Style mới)
   ======================= */
.panel {
  background: #fff;
  border-radius: 14px;
  padding: 16px;
  border: 1px solid rgba(255, 77, 79, 0.18);
  box-shadow: 0 18px 50px rgba(17, 24, 39, 0.08);
  margin-bottom: 20px;
}

.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 12px;
}

.toolbar-left,
.toolbar-right {
  display: flex;
  align-items: center;
  gap: 10px;
}

.search-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.search-icon {
  position: absolute;
  left: 12px;
  color: rgba(17, 24, 39, 0.40);
  font-size: 13px;
  pointer-events: none;
}

.search-input {
  height: 36px;
  padding: 0 12px 0 34px;
  border-radius: 12px;
  border: 1px solid rgba(17, 24, 39, 0.14);
  outline: none;
  min-width: 420px;
  color: rgba(17, 24, 39, 0.82);
  font-size: 13px;
  background: #F9FAFB;
}

.search-input:focus {
  border-color: rgba(255, 77, 79, 0.45);
  background: #fff;
  box-shadow: 0 0 0 3px rgba(255, 77, 79, 0.10);
}

.btn {
  height: 36px;
  padding: 0 14px;
  border-radius: 10px;
  border: 1px solid rgba(17, 24, 39, 0.14);
  background: #fff;
  color: rgba(17, 24, 39, 0.88);
  font-size: 13px;
  font-weight: 400;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  transition: 0.15s ease;
}

.btn-mi {
  font-size: 18px;
  line-height: 1;
}

.btn-reset {
  background: #4b5563 !important;
  color: #fff !important;
  border: none !important;
}

.btn-reset:hover {
  filter: brightness(0.98);
}

.btn-newaccount {
  border: none !important;
  color: #fff !important;
  height: 36px;
  padding: 0 14px;
  border-radius: 10px;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  cursor: pointer;
  background: linear-gradient(90deg, #ff4d4f 0%, #111827 100%) !important;
  box-shadow: 0 10px 18px rgba(255, 77, 79, 0.16);
}

.btn-newaccount:hover {
  filter: brightness(0.98);
}

.filters-bar {
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
  margin-top: 8px;
}

.filter-group {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  white-space: nowrap;
}

.ss-label {
  white-space: nowrap;
  margin: 0;
  line-height: 1;
  font-size: 13px;
  font-weight: 400;
  color: rgba(17, 24, 39, 0.82);
  flex: 0 0 auto;
}

.filter-pill {
  height: 36px;
  min-width: 160px;
  border-radius: 12px;
  font-size: 13px;
  font-weight: 400;
  color: rgba(17, 24, 39, 0.82);
  border: 1px solid rgba(17, 24, 39, 0.14);
  padding: 0 10px;
  outline: none;
  background: #fff;
}

.filter-pill:focus {
  border-color: rgba(255, 77, 79, 0.45);
}

@media (max-width: 900px) {
  .search-input {
    min-width: 260px;
    width: 100%;
  }

  .filters-bar {
    gap: 10px;
  }
}
</style>