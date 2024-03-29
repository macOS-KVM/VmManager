<template>
  <q-dialog v-model="layout">
    <q-layout
      view="hHh lpR fFf"
      container
      :class="{ 'bg-dark': $q.dark.isActive, 'bg-white': !$q.dark.isActive }"
    >
      <q-header bordered>
        <q-toolbar>
          <q-toolbar-title>Create VM</q-toolbar-title>
          <q-btn icon="close" flat round dense v-close-popup />
        </q-toolbar>

        <q-tabs align="left" v-model="tab">
          <q-tab name="general" label="General" />
          <q-tab name="memory" label="Memory" />
          <q-tab name="disk" label="Disk" />
          <q-tab name="cdrom" label="CD-ROM" />
          <q-tab name="network" label="Network" />
        </q-tabs>
        <q-separator color="transparent" />
      </q-header>

      <q-page-container>
        <q-page padding>
          <q-tab-panel name="general" v-show="tab == 'general'">
            <q-input
              label="Name"
              v-model="general_name"
              :rules="[
                (val) =>
                  (val && val.length > 0) ||
                  'Name needs to be at least 1 character long',
              ]"
            />
            <q-select label="OS" v-model="general_os" :options="osOptions" />
            <q-select
              label="Machine"
              v-model="general_machine"
              :options="machineOptions"
            />
            <q-select
              label="BIOS"
              v-model="general_bios"
              :options="biosOptions"
            />
            <q-select
              label="OVMF"
              v-model="general_ovmf"
              option-label="name"
              :options="ovmfOptions"
              v-if="general_bios == 'ovmf'"
            >
              <template v-slot:option="scope">
                <q-item v-bind="scope.itemProps">
                  <q-item-section>
                    <q-item-label>{{ scope.opt.name }}</q-item-label>
                    <q-item-label caption
                      >Path: {{ scope.opt.path }}</q-item-label
                    >
                  </q-item-section>
                </q-item>
              </template>
            </q-select>
          </q-tab-panel>

          <q-tab-panel name="memory" v-show="tab == 'memory'">
            <div class="row">
              <div class="col">
                <q-input
                  label="Memory minimum"
                  v-model="memory_minMemory"
                  type="number"
                  min="1"
                />
              </div>
              <div class="col-md-auto">
                <q-select
                  v-model="memory_minMemoryUnit"
                  :options="memoryUnitOptions"
                />
              </div>
            </div>

            <div class="row">
              <div class="col">
                <q-input
                  label="Memory maximum"
                  v-model="memory_maxMemory"
                  type="number"
                  min="1"
                />
              </div>
              <div class="col-md-auto">
                <q-select
                  v-model="memory_maxMemoryUnit"
                  :options="memoryUnitOptions"
                />
              </div>
              <q-space />
            </div>
          </q-tab-panel>

          <q-tab-panel name="disk" v-show="tab == 'disk'">
            <div class="row">
              <div class="col">
                <q-input
                  label="Disk size"
                  v-model="disk_size"
                  type="number"
                  min="1"
                />
              </div>
              <div class="col-md-auto">
                <q-select v-model="disk_size_unit" :options="diskUnitOptions" />
              </div>
            </div>
            <div class="row">
              <div class="col">
                <q-select
                  label="Disk type"
                  v-model="disk_type"
                  :options="diskTypeOptions"
                />
              </div>
            </div>
            <div class="row">
              <div class="col">
                <q-select
                  label="Disk bus"
                  v-model="disk_bus"
                  :options="diskBusOptions"
                />
              </div>
            </div>
            <div class="row">
              <div class="col">
                <DirectoryList
                  v-model="disk_location"
                  label="Disk Location"
                  selectiontype="dir"
                />
              </div>
            </div>
          </q-tab-panel>
          <q-tab-panel name="cdrom" v-show="tab == 'cdrom'">
            <DirectoryList v-model="cdrom_location" label="CD-ROM Location" />
            <q-select
              label="Bus"
              v-model="cdrom_bus"
              :options="cdromBusOptions"
            />
          </q-tab-panel>
          <q-tab-panel name="network" v-show="tab == 'network'">
            <NetworkList ref="networkSource" />
            <q-select
              label="Model"
              v-model="network_model"
              :options="networkModelOptions"
            />
          </q-tab-panel>
        </q-page>
        <q-footer reveal bordered>
          <q-toolbar>
            <q-btn
              flat
              icon="mdi-arrow-left"
              @click="prevTabPanel()"
              v-if="tab != 'general'"
            />
            <q-space />
            <q-btn
              flat
              label="Finish"
              @click="createVm()"
              v-if="tab == 'network'"
            />
            <q-btn flat icon="mdi-arrow-right" @click="nextTabPanel()" v-else />
          </q-toolbar>
        </q-footer>
      </q-page-container>
    </q-layout>
  </q-dialog>
  <ErrorDialog ref="errorDialog" />
</template>

<script>
import { ref } from "vue";
import ErrorDialog from "src/components/ErrorDialog.vue";
import DirectoryList from "src/components/DirectoryList.vue";
import NetworkList from "src/components/NetworkList.vue";

export default {
  data() {
    return {
      layout: ref(false),
      tab: ref("general"),
      osOptions: [
        "Microsoft Windows 11",
        "Microsft Windows 10",
        "Microsoft Windows 8.1",
        "Microsoft Windows 8",
        "Microsoft Windows 7",
        "macOS 10.15 Catalina",
        "macOS 11 Big Sur",
        "macOS 12 Monterey",
        "macOS 13 Ventura",
        "Linux",
      ],
      machineOptions: [],
      biosOptions: ["ovmf", "BIOS"],
      memoryMinOptions: [
        "1024",
        "2048",
        "4096",
        "8192",
        "16384",
        "32768",
        "65536",
      ],
      memoryUnitOptions: ["MB", "GB", "TB"],
      general_name: ref("New Virtual Machine"),
      general_os: ref("Microsoft Windows 10"),
      general_machine: ref(),
      general_bios: ref("ovmf"),
      general_ovmf: ref(),
      ovmfOptions: [],
      memory_minMemory: ref(1024),
      memory_minMemoryUnit: ref("MB"),
      memory_maxMemory: ref(1024),
      memory_maxMemoryUnit: ref("MB"),
      disk_size: ref(10),
      disk_size_unit: ref("GB"),
      diskUnitOptions: ["MB", "GB", "TB"],
      diskTypeOptions: ["raw", "qcow2"],
      disk_type: ref("raw"),
      disk_bus: ref("sata"),
      disk_location: "",
      diskBusOptions: ["sata", "scsi", "virtio", "usb"],
      cdrom_bus: ref("sata"),
      cdrom_location: "",
      cdromBusOptions: ["sata", "scsi", "virtio", "usb"],
      network_model: ref("virtio"),
      networkModelOptions: ["virtio", "e1000", "rtl8139"],
    };
  },
  components: {
    ErrorDialog,
    DirectoryList,
    NetworkList,
  },
  methods: {
    show() {
      this.layout = true;
      this.getMachineTypes();
      this.getOvmf();
    },
    getMachineTypes() {
      this.$api
        .get("/host/system-info/guest-machine-types")
        .then((response) => {
          this.machineOptions = response.data;
          this.general_machine =
            this.machineOptions[this.machineOptions.length - 1];
        })
        .catch((error) => {
          this.$refs.errorDialog.show("Failed to get machine types", [
            error.response.data.detail,
          ]);
        });
    },
    getOvmf() {
      this.$api.get("/vm-manager/settings/ovmf-paths/all").then((response) => {
        console.log(response.data);
        this.ovmfOptions = response.data;
        this.general_ovmf = this.ovmfOptions[0];
      });
    },
    createVm() {
      if (this.disk_location == null) {
        this.$refs.errorDialog.show("Invalid disk location", [
          "Please a valid directory where the vm disk can be stored",
        ]);
        return;
      }

      if (this.cdrom_location == null) {
        this.$refs.errorDialog.show("Invalid cdrom location", [
          "Please a valid directory where the vm cdrom can be stored",
        ]);
        return;
      }
      if (this.$refs.networkSource.getSelectedNetwork() == null) {
        this.$refs.errorDialog.show(
          "Please select a network"[
            ("Please select a network",
            "Create a network if you don't have one")
          ],
        );
        return;
      }

      const formData = new FormData();
      formData.append("name", this.general_name);
      formData.append("os", this.general_os);
      formData.append("machine_type", this.general_machine);
      formData.append("bios_type", this.general_bios);
      formData.append("ovmf_name", this.general_ovmf.name);
      formData.append("memory_min", this.memory_minMemory);
      formData.append("memory_min_unit", this.memory_minMemoryUnit);
      formData.append("memory_max", this.memory_maxMemory);
      formData.append("memory_max_unit", this.memory_maxMemoryUnit);
      formData.append("disk_size", this.disk_size);
      formData.append("disk_size_unit", this.disk_size_unit);
      formData.append("disk_type", this.disk_type);
      formData.append("disk_bus", this.disk_bus);
      formData.append("disk_location", this.disk_location);
      formData.append("cdrom_location", this.cdrom_location);
      formData.append(
        "network_source",
        this.$refs.networkSource.getSelectedNetwork(),
      );
      formData.append("network_model", this.network_model);
      this.$api
        .post("/vm-manager/create", formData)
        .then((this.layout = false))
        .catch((error) => {
          this.$refs.errorDialog.show("Error creating VM", [
            error.response.data.detail,
          ]);
        });
    },
    prevTabPanel() {
      if (this.tab === "memory") {
        this.tab = "general";
      } else if (this.tab === "disk") {
        this.tab = "memory";
      } else if (this.tab === "cdrom") {
        this.tab = "disk";
      } else if (this.tab === "network") {
        this.tab = "cdrom";
      }
    },
    nextTabPanel() {
      if (this.tab === "general") {
        this.tab = "memory";
      } else if (this.tab === "memory") {
        this.tab = "disk";
      } else if (this.tab === "disk") {
        this.tab = "cdrom";
      } else if (this.tab === "cdrom") {
        this.tab = "network";
      }
    },
  },
};
</script>
