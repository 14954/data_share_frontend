<template>
  <v-container fluid>
    <!-- 页面标题 -->
    <v-row>
      <v-col cols="12">
        <div class="d-flex align-center justify-space-between mb-4">
          <div class="d-flex align-center">
            <v-icon size="40" color="primary" class="mr-3"
              >mdi-folder-account</v-icon
            >
            <div>
              <h1 class="text-h4 font-weight-bold">我的数据</h1>
              <p class="text-subtitle-1 text-grey mb-0">管理您的数据集</p>
            </div>
          </div>
          <v-btn
            color="primary"
            size="large"
            :loading="loading"
            @click="load"
          >
            <v-icon start>mdi-refresh</v-icon>
            刷新
          </v-btn>
        </div>
      </v-col>
    </v-row>

    <!-- 1.我登记的数据 -->
    <v-card class="mb-6" elevation="2">
      <v-card-title class="bg-primary text-white d-flex align-center">
        <v-icon class="mr-2">mdi-file-document-multiple</v-icon>
        我登记的数据
      </v-card-title>

      <v-card-text class="pa-0">
        <!-- 空状态 -->
        <div
          v-if="!loading && owned.length === 0"
          class="text-center pa-12"
        >
          <v-icon size="80" color="grey-lighten-1"
            >mdi-folder-open-outline</v-icon
          >
          <h3 class="text-h6 mt-4 text-grey">暂无数据</h3>
          <p class="text-body-2 text-grey mt-2">
            您还没有登记任何数据集，去登记一个吧！
          </p>
          <v-btn color="primary" class="mt-2" to="/main/register-data">
            <v-icon start>mdi-plus</v-icon>
            登记数据
          </v-btn>
        </div>

        <!-- 数据列表 -->
        <v-list v-else lines="three">
          <template v-for="(d, index) in owned" :key="d.id">
            <v-list-item>
              <template v-slot:prepend>
                <v-avatar color="primary" size="48">
                  <v-icon color="white">mdi-file-document</v-icon>
                </v-avatar>
              </template>

              <v-list-item-title class="text-h6 font-weight-medium mb-2">
                {{ d.name }}
              </v-list-item-title>

              <v-list-item-subtitle class="mb-2">
                <v-chip
                  size="small"
                  color="info"
                  variant="tonal"
                  class="mr-2"
                >
                  <v-icon start size="small">mdi-tag</v-icon>
                  {{ d.domain }}
                </v-chip>
                <v-chip
                  size="small"
                  color="warning"
                  variant="tonal"
                  class="mr-2"
                >
                  <v-icon start size="small">mdi-file-code</v-icon>
                  {{ d.dataType }}
                </v-chip>
                <v-chip size="small" color="success" variant="tonal">
                  <v-icon start size="small">mdi-download</v-icon>
                  {{ d.downloads }} 次下载
                </v-chip>
              </v-list-item-subtitle>

              <v-list-item-subtitle class="text-grey-darken-1">
                {{ d.description || "暂无描述" }}
              </v-list-item-subtitle>

              <template v-slot:append>
                <div class="d-flex flex-column align-center">
                  <v-switch
                    v-model="d.isListed"
                    color="success"
                    :loading="d.updating"
                    hide-details
                    @update:model-value="(val) => toggle(d, val)"
                  >
                    <template v-slot:label>
                      <span :class="d.isListed ? 'text-success' : 'text-grey'">
                        {{ d.isListed ? "已上架" : "未上架" }}
                      </span>
                    </template>
                  </v-switch>
                </div>
              </template>
            </v-list-item>
            <v-divider v-if="index < owned.length - 1" :key="`divider-${d.id}`"></v-divider>
          </template>
        </v-list>
      </v-card-text>

      <!-- 加载中 -->
      <v-overlay
        :model-value="loading"
        contained
        class="align-center justify-center"
      >
        <v-progress-circular
          color="primary"
          indeterminate
          size="64"
        ></v-progress-circular>
      </v-overlay>
    </v-card>

    <!-- 2. 我共享的数据 -->
    <v-card class="mb-6" elevation="2">
      <v-card-title class="bg-primary text-white d-flex align-center">
        <v-icon class="mr-2">mdi-file-document-multiple</v-icon>
        我共享的数据
      </v-card-title>

      <v-card-text class="pa-0">
        <!-- 空状态 -->
        <div
          v-if="!loading && sharings.length === 0"
          class="text-center pa-12"
        >
          <v-icon size="80" color="grey-lighten-1"
            >mdi-folder-open-outline</v-icon
          >
          <h3 class="text-h6 mt-4 text-grey">暂无数据</h3>
          <p class="text-body-2 text-grey mt-2">
            您还没有共享任何数据！
          </p>
          <!-- <v-btn color="primary" class="mt-2" to="/register-data">
            <v-icon start>mdi-plus</v-icon>
            共享数据
          </v-btn> -->
        </div>

        <!-- 数据列表 -->
        <v-list v-if="sharings.length">
          <v-list-item v-for="sharing in sharings" :key="sharing.id">
            <v-list-item-title class="font-weight-medium">
              {{ sharing.datasetName }}
            </v-list-item-title>

            <v-list-item-subtitle>
              请求人：{{ sharing.consumerName}}
              <span class="ml-2 text-grey">理由：{{ sharing.request_description || "无" }}</span>
            </v-list-item-subtitle>

            <template #append>
              <div class="d-flex ga-2 align-center">
                <v-chip
                  v-if="sharing.status === 'approved'"
                  size="small"
                  color="success"
                  variant="tonal"
                >
                  已批准
                </v-chip>

                <v-chip
                  v-else-if="sharing.status === 'rejected'"
                  size="small"
                  color="error"
                  variant="tonal"
                >
                  已拒绝
                </v-chip>

                <div v-else class="d-flex ga-2">
                  <v-btn color="success" variant="tonal" size="small"
                    :loading="sharing._busy"
                    @click="decide(sharing, true)">
                    批准
                  </v-btn>
                  <v-btn color="error" variant="tonal" size="small"
                    :loading="sharing._busy"
                    @click="decide(sharing, false)">
                    拒绝
                  </v-btn>
                </div>
              </div>
            </template>
          </v-list-item>
        </v-list>

        <v-alert v-else type="info" variant="tonal">暂无共享请求</v-alert>

      </v-card-text>

      <!-- 加载中 -->
      <v-overlay
        :model-value="loading"
        contained
        class="align-center justify-center"
      >
        <v-progress-circular
          color="primary"
          indeterminate
          size="64"
        ></v-progress-circular>
      </v-overlay>
    </v-card>

    <v-dialog v-model="showApproveDialog" max-width="760">
      <v-card>
        <v-card-title class="bg-primary text-white">生成临时下载链接</v-card-title>
        <v-card-text class="pa-6">
          <v-text-field v-model="approveRegion" label="Region" :rules="approveRegionRules" required />
          <v-text-field v-model="approveBucket" label="Bucket" :rules="approveBucketRules" required />
          <v-text-field v-model="approveEndpoint" label="Endpoint（可选）" :rules="approveEndpointRules" />
          <v-text-field
            v-model="approveAccessKeyId"
            :type="showApproveAccessKey ? 'text' : 'password'"
            label="Access Key ID"
            :append-icon="showApproveAccessKey ? 'mdi-eye-off' : 'mdi-eye'"
            @click:append="showApproveAccessKey = !showApproveAccessKey"
            :rules="approveAccessKeyRules"
            required
          />
          <v-text-field
            v-model="approveSecretAccessKey"
            :type="showApproveSecretKey ? 'text' : 'password'"
            label="Secret Access Key"
            :append-icon="showApproveSecretKey ? 'mdi-eye-off' : 'mdi-eye'"
            @click:append="showApproveSecretKey = !showApproveSecretKey"
            :rules="approveSecretKeyRules"
            required
          />
          <v-text-field
            v-model="approveSessionToken"
            :type="showApproveSessionToken ? 'text' : 'password'"
            label="Session Token"
            :append-icon="showApproveSessionToken ? 'mdi-eye-off' : 'mdi-eye'"
            @click:append="showApproveSessionToken = !showApproveSessionToken"
            :rules="approveSessionTokenRules"
            required
          />
          <v-textarea
            v-model="approveCredentialJson"
            label="JSON 凭证（可粘贴 STS JSON 并解析）"
            rows="3"
            clearable
          />
          <v-btn variant="text" size="small" @click="parseApproveCredentialJson">解析凭证</v-btn>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn variant="text" @click="closeApproveDialog">取消</v-btn>
          <v-btn color="primary" :loading="approveSubmitting" :disabled="!canSubmitApproval" @click="confirmGenerateUrl">
            确认生成临时下载链接
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

<!-- 3.共享给我的数据 -->
    <v-card class="mb-6" elevation="2">
      <v-card-title class="bg-primary text-white d-flex align-center">
        <v-icon class="mr-2">mdi-file-document-multiple</v-icon>
        共享给我的数据
      </v-card-title>

      <v-card-text class="pa-0">
        <!-- 空状态 -->
        <div
          v-if="!loading && shareds.length === 0"
          class="text-center pa-12"
        >
          <v-icon size="80" color="grey-lighten-1"
            >mdi-folder-open-outline</v-icon
          >
          <h3 class="text-h6 mt-4 text-grey">暂无数据</h3>
          <p class="text-body-2 text-grey mt-2">
            您还没有被共享任何数据！
          </p>
          <v-btn color="primary" class="mt-2" to="/main/market">
            <v-icon start>mdi-plus</v-icon>
            数据查找
          </v-btn>
        </div>
        
        <!-- 数据列表 -->
        <v-list v-if="shareds.length" lines="two">
          <v-list-item v-for="shared in shareds" :key="shared.id">
            <v-list-item-title class="font-weight-medium">
              {{ shared.datasetName }}
            </v-list-item-title>
            <v-list-item-subtitle>
              来自：{{ shared.providerName}}
            </v-list-item-subtitle>

            <template #append>
              <v-btn
                color="primary"
                variant="tonal"
                :loading="downloading"
                @click="download(shared)"
              >
                <v-icon start size="large">mdi-download</v-icon>
                下载
              </v-btn>
            </template>
          </v-list-item>
        </v-list>

        <v-alert v-else type="info" variant="tonal">暂无共享给我的数据</v-alert>
      </v-card-text>

      <!-- 加载中 -->
      <v-overlay
        :model-value="loading"
        contained
        class="align-center justify-center"
      >
        <v-progress-circular
          color="primary"
          indeterminate
          size="64"
        ></v-progress-circular>
      </v-overlay>
    </v-card>

    <!-- 4. 我发起的共享请求 -->
    <v-card class="mb-6" elevation="2">
      <v-card-title class="bg-primary text-white d-flex align-center">
        <v-icon class="mr-2">mdi-send</v-icon>
        我发起的共享请求
      </v-card-title>

      <v-card-text class="pa-0">
        <div v-if="!loading && requestsByMe.length === 0" class="text-center pa-12">
          <v-icon size="80" color="grey-lighten-1">mdi-send</v-icon>
          <h3 class="text-h6 mt-4 text-grey">暂无请求</h3>
          <p class="text-body-2 text-grey mt-2">您还没有发起任何共享请求。</p>
        </div>

        <v-list v-if="requestsByMe.length" lines="two">
          <v-list-item v-for="req in requestsByMe" :key="req.id">
            <v-list-item-title class="font-weight-medium">{{ req.datasetName || '未知数据' }}</v-list-item-title>
            <v-list-item-subtitle>
              提供方：{{ req.providerName || '未知' }}
              <span class="ml-2 text-grey">说明：{{ req.request_description || '无' }}</span>
            </v-list-item-subtitle>

            <template #append>
              <v-chip size="small" :color="req.status === 'approved' ? 'success' : (req.status === 'rejected' ? 'error' : 'grey')" variant="tonal">
                {{ req.status === 'approved' ? '已共享' : (req.status === 'rejected' ? '已拒绝' : '待处理') }}
              </v-chip>

              <v-btn
                v-if="req.status === 'approved'"
                color="primary"
                variant="tonal"
                size="small"
                class="ml-2"
                :loading="downloading"
                @click="download(req)"
              >
                <v-icon start>mdi-download</v-icon>
                下载
              </v-btn>
            </template>
          </v-list-item>
        </v-list>

        <v-alert v-else type="info" variant="tonal">暂无发起的请求</v-alert>
      </v-card-text>

      <v-overlay :model-value="loading" contained class="align-center justify-center">
        <v-progress-circular color="primary" indeterminate size="64"></v-progress-circular>
      </v-overlay>
    </v-card>




    <!-- 我下载的数据 -->
    <v-card elevation="2">
      <v-card-title class="bg-secondary text-white d-flex align-center">
        <v-icon class="mr-2">mdi-download-multiple</v-icon>
        我下载的数据
      </v-card-title>

      <v-card-text>
        <v-alert type="info" variant="tonal" prominent>
          <v-alert-title class="text-h6">功能说明</v-alert-title>
          <div class="mt-2">
            <!-- 后续可用 download_logs 按 userId 聚合展示已下载的数据列表。 -->
          </div>
        </v-alert>
      </v-card-text>
    </v-card>

    <!-- 错误提示 -->
    <v-snackbar v-model="showError" color="error" :timeout="3000">
      <v-icon start>mdi-alert-circle</v-icon>
      {{ err }}
    </v-snackbar>
  </v-container>
</template>

<script setup>
import { computed, onMounted, ref, watch } from "vue";
import { api } from "../api";

const APPROVE_PREF_KEY = "myData.shareApprove.s3.publicPrefs";

const owned = ref([]);
const err = ref("");
const loading = ref(false);
const showError = ref(false);
const sharings = ref([]);
const shareds = ref([]);
const requestsByMe = ref([]);
const downloading = ref(false);
const showApproveDialog = ref(false);
const approveSubmitting = ref(false);
const pendingShare = ref(null);
const approveRegion = ref("");
const approveBucket = ref("");
const approveEndpoint = ref("");
const approveAccessKeyId = ref("");
const approveSecretAccessKey = ref("");
const approveSessionToken = ref("");
const approveCredentialJson = ref("");
const showApproveAccessKey = ref(false);
const showApproveSecretKey = ref(false);
const showApproveSessionToken = ref(false);

const approveRegionRules = [(v) => !!String(v || "").trim() || "请输入 Region"];
const approveBucketRules = [(v) => !!String(v || "").trim() || "请输入 Bucket"];
const approveAccessKeyRules = [(v) => !!String(v || "").trim() || "请输入 Access Key ID"];
const approveSecretKeyRules = [(v) => !!String(v || "").trim() || "请输入 Secret Access Key"];
const approveSessionTokenRules = [(v) => !!String(v || "").trim() || "请输入 Session Token"];
const approveEndpointRules = [(v) => !v || isValidHttpUrl(v) || "Endpoint 必须是 http/https 地址"];
const canSubmitApproval = computed(() =>
  !!pendingShare.value
  && !!approveRegion.value.trim()
  && !!approveBucket.value.trim()
  && !!approveAccessKeyId.value.trim()
  && !!approveSecretAccessKey.value.trim()
  && !!approveSessionToken.value.trim()
  && !approveSubmitting.value
);

function isValidHttpUrl(u) {
  try {
    const p = new URL(u);
    return p.protocol === "http:" || p.protocol === "https:";
  } catch {
    return false;
  }
}

function loadApprovePrefs() {
  try {
    const raw = localStorage.getItem(APPROVE_PREF_KEY);
    if (!raw) return;
    const p = JSON.parse(raw);
    approveRegion.value = p.region || "";
    approveBucket.value = p.bucket || "";
    approveEndpoint.value = p.endpoint || "";
  } catch {
    localStorage.removeItem(APPROVE_PREF_KEY);
  }
}

function persistApprovePrefs() {
  localStorage.setItem(APPROVE_PREF_KEY, JSON.stringify({
    region: approveRegion.value.trim(),
    bucket: approveBucket.value.trim(),
    endpoint: approveEndpoint.value.trim(),
  }));
}

function clearApproveCredentials() {
  approveAccessKeyId.value = "";
  approveSecretAccessKey.value = "";
  approveSessionToken.value = "";
  approveCredentialJson.value = "";
  showApproveAccessKey.value = false;
  showApproveSecretKey.value = false;
  showApproveSessionToken.value = false;
}

function closeApproveDialog() {
  showApproveDialog.value = false;
  pendingShare.value = null;
  clearApproveCredentials();
}

function parseApproveCredentialJson() {
  if (!approveCredentialJson.value.trim()) {
    err.value = "请输入 JSON 凭证内容";
    showError.value = true;
    return;
  }
  try {
    const parsed = JSON.parse(approveCredentialJson.value);
    const accessV = parsed.accessKeyId || parsed.AccessKeyId || parsed.aws_access_key_id;
    const secretV = parsed.secretAccessKey || parsed.SecretAccessKey || parsed.aws_secret_access_key;
    const tokenV = parsed.sessionToken || parsed.SessionToken || parsed.Token || parsed.aws_session_token;
    if (!accessV || !secretV || !tokenV) {
      throw new Error("JSON 缺少必要字段：accessKeyId、secretAccessKey、sessionToken");
    }
    approveAccessKeyId.value = String(accessV).trim();
    approveSecretAccessKey.value = String(secretV).trim();
    approveSessionToken.value = String(tokenV).trim();
    err.value = "";
    showError.value = false;
  } catch (e) {
    err.value = e?.message || "JSON 解析失败";
    showError.value = true;
  }
}

function openApproveDialog(sharing) {
  pendingShare.value = sharing;
  approveSubmitting.value = false;
  clearApproveCredentials();
  showApproveDialog.value = true;
}

async function confirmGenerateUrl() {
  if (!canSubmitApproval.value) return;
  approveSubmitting.value = true;
  try {
    persistApprovePrefs();
    const { data } = await api.post("/local/generate_url", {
      shareId: pendingShare.value.id,
      objectKey: pendingShare.value.objectKey,
      bucket: approveBucket.value.trim(),
      access_key_id: approveAccessKeyId.value.trim(),
      secret_access_key: approveSecretAccessKey.value.trim(),
      session_token: approveSessionToken.value.trim(),
      region: approveRegion.value.trim(),
      endpoint: approveEndpoint.value.trim() || undefined,
    });
    pendingShare.value.status = "approved";
    pendingShare.value.signedUrl = data?.signed_url || "";
    showApproveDialog.value = false;
    clearApproveCredentials();
    pendingShare.value = null;
  } catch (e) {
    err.value = e?.response?.data?.details || e?.response?.data?.message || "生成临时下载链接失败";
    showError.value = true;
  } finally {
    approveSubmitting.value = false;
  }
}

async function fetchSharing() {
  const { data } = await api.get("/remote/shares/sharing-with-others");
  sharings.value = (data.sharing || []).map(x => ({
    ...x,
    _busy: false,
  }));
}

async function decide(r, statusBool) {
  if (statusBool) {
    openApproveDialog(r);
    return;
  }
  r._busy = true;
  try {
    await api.post("/remote/shares/reject", { shareId: r.id });
    r.status = "rejected";
    r.responded_at = new Date().toISOString();
  } catch (e) {
    err.value = e?.response?.data?.message || "操作失败";
    showError.value = true;
  } finally {
    r._busy = false;
  }
}
onMounted(fetchSharing);

async function fetchSharedWithMe() {
  const { data } = await api.get("/remote/shares/shared-with-me");
  shareds.value = (data.shared || []).map(x => ({ ...x }));
  downloading.value = false;
}

function normalizeStorageType(record) {
  const raw = (record?.storage_type ?? record?.storageType ?? "").toString().trim().toLowerCase();
  if (raw === "local" || raw === "s3") return raw;
  return "";
}

function parseDownloadFileName(contentDisposition) {
  if (!contentDisposition) return "";

  const encoded = contentDisposition.match(/filename\*=UTF-8''([^;]+)/i);
  if (encoded?.[1]) {
    try {
      return decodeURIComponent(encoded[1]);
    } catch {
      return encoded[1];
    }
  }

  const quoted = contentDisposition.match(/filename="([^"]+)"/i);
  if (quoted?.[1]) return quoted[1];

  const plain = contentDisposition.match(/filename=([^;]+)/i);
  return plain?.[1]?.trim() || "";
}

function downloadBlob(data, fileName, contentType) {
  const blob = new Blob([data], { type: contentType || "application/octet-stream" });
  const url = window.URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = fileName || "dataset";
  document.body.appendChild(a);
  a.click();
  a.remove();
  window.URL.revokeObjectURL(url);
}

async function download(r) {
  err.value = "";
  downloading.value = true;
  try {
    const datasetId = r?.datasetId;
    if (!datasetId) {
      throw new Error("缺少 datasetId，无法下载");
    }

    const storageType = normalizeStorageType(r);
    if (!storageType) {
      throw new Error("缺少 storage_type，无法判断下载方式");
    }

    if (storageType === "local") {
      const resp = await api.get(`/remote/datasets/${datasetId}/download`, {
        responseType: "blob",
      });
      const headerFileName = parseDownloadFileName(resp.headers?.["content-disposition"]);
      downloadBlob(resp.data, headerFileName || `${r.datasetName || "dataset"}`, resp.headers?.["content-type"]);
      return;
    }

    const inlineSignedUrl = r?.signedUrl || r?.signed_url;
    if (inlineSignedUrl) {
      window.open(inlineSignedUrl, "_blank", "noopener");
      return;
    }

    const { data } = await api.get(`/remote/datasets/${datasetId}/download-url`);
    const downloadUrl = data?.downloadUrl;
    if (!downloadUrl) {
      throw new Error("下载链接无效");
    }
    window.open(downloadUrl, "_blank", "noopener");
  } catch (e) {
    err.value = e?.response?.data?.message || e?.message || "下载失败";
    showError.value = true;
    console.error("download error", e);
  } finally {
    downloading.value = false;
  }
}
onMounted(fetchSharedWithMe);

async function fetchRequestsByMe() {
  try {
    const { data } = await api.get('/remote/shares/requests-by-me');
    requestsByMe.value = (data.requests || []).map(x => ({
      ...x,
      status: x.status,
    }));
  } catch (e) {
    console.error('fetchRequestsByMe error', e);
  }
}
onMounted(fetchRequestsByMe);

async function load() {
  err.value = "";
  loading.value = true;
  try {
    const { data } = await api.get("/remote/datasets/mine");
    owned.value = (data.owned || []).map((item) => ({
      ...item,
      updating: false,
    }));
  } catch (e) {
    err.value = e?.response?.data?.message || "加载失败";
    showError.value = true;
  } finally {
    loading.value = false;
  }
}

async function toggle(d, isListed) {
  d.updating = true;
  try {
    const { data } = await api.patch(`/remote/datasets/${d.id}/listing`, {
      isListed,
    });
    d.isListed = data.dataset.isListed;
  } catch (e) {
    err.value = e?.response?.data?.message || "更新失败";
    showError.value = true;
    d.isListed = !isListed;
  } finally {
    d.updating = false;
  }
}

watch(showError, (val) => {
  if (!val) err.value = "";
});

watch([approveRegion, approveBucket, approveEndpoint], () => {
  persistApprovePrefs();
});

loadApprovePrefs();
onMounted(load);
</script>
