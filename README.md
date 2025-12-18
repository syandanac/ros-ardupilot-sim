# Tutorial SITL + ArduPilot + Gazebo + ROS Camera Plugin

Tutorial ini memberikan panduan lengkap bagi kita untuk menyiapkan simulasi drone menggunakan **ArduPilot (ArduCopter)** dan **SITL** **(Software in the Loop)** dalam lingkungan *virtual* 3D **Gazebo**. Bagian penting dari tutorial ini adalah integrasi *plugin* kamera **ROS** untuk mempublikasikan *image feed* dari simulasi ke dalam ekosistem ROS.

## Persyaratan Sistem

* **Ubuntu 20.04 LTS** (Sangat direkomendasikan menggunakan grafis 3D penuh).
* **Gazebo versi 11.x**.
* **ROS Noetic**.
* **MAVROS**.

---

## 1. Instalasi Gazebo 11

### Konfigurasi Repositori

Kita perlu mengatur komputer agar dapat menerima perangkat lunak dari `packages.osrfoundation.org`:

```bash
sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -

```

### Instalasi Perangkat Lunak

```bash
sudo apt-get update
sudo apt-get install gazebo11 libgazebo-dev

```

### Pengujian Gazebo

```bash
gazebo --verbose

```

> **Catatan:** Jika kita menggunakan *Virtual Machine* (VM), mungkin perlu menonaktifkan *hardware acceleration* tertentu agar Gazebo berjalan lancar:
> `echo "export SVGA_VGPU10=0" >> ~/.profile`

---

## 2. Instalasi ROS Noetic

### Konfigurasi Sumber Perangkat Lunak

```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

sudo apt update
sudo apt install ros-noetic-desktop-full

```

### Inisialisasi *Environment*

```bash
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc

```

### Instalasi Dependensi *Build*

```bash
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
sudo rosdep init
rosdep update

```

---

## 3. Instalasi MAVROS

*MAVROS* adalah *node* komunikasi *MAVLink* yang menghubungkan ROS dengan *Flight Controller* (ArduPilot).

```bash
sudo apt-get install ros-noetic-mavros ros-noetic-mavros-extras
wget https://raw.githubusercontent.com/mavlink/mavros/master/mavros/scripts/install_geographiclib_datasets.sh
chmod a+x install_geographiclib_datasets.sh
./install_geographiclib_datasets.sh

```

### Instalasi Catkin Tools & RQT

```bash
sudo apt-get install ros-noetic-rqt ros-noetic-rqt-common-plugins ros-noetic-rqt-robot-plugins
sudo apt-get install python3-catkin-tools

```

---

## 4. Pengaturan ArduPilot SITL

### *Clone* Repositori ArduPilot

Kita akan mengunduh *source code* ArduPilot:

```bash
git clone --recurse-submodules https://github.com/ArduPilot/ardupilot.git
cd ardupilot

```

### Instalasi Prasyarat

Kita jalankan skrip bantuan untuk menginstal semua *libraries* yang diperlukan:

```bash
Tools/environment_install/install-prereqs-ubuntu.sh -y
. ~/.profile

```

### *Compilation* SITL

```bash
./waf configure --board sitl
./waf copter

```

---

## 5. Instalasi Plugin Gazebo ArduPilot

Agar Gazebo dapat berkomunikasi dengan ArduPilot, kita memerlukan *plugin* khusus bernama `ardupilot_gazebo`.

```bash
cd ~/
git clone https://github.com/r0ch1n/ardupilot_gazebo
cd ardupilot_gazebo
mkdir build && cd build
cmake ..
make -j4
sudo make install

```

### Konfigurasi Environment Path

Tambahkan konfigurasi berikut ke dalam `~/.bashrc` agar Gazebo mengenali model kita:

```bash
echo 'source /usr/share/gazebo/setup.sh' >> ~/.bashrc
echo 'export GAZEBO_MODEL_PATH=~/ardupilot_gazebo/models' >> ~/.bashrc
echo 'export GAZEBO_RESOURCE_PATH=~/ardupilot_gazebo/worlds:${GAZEBO_RESOURCE_PATH}' >> ~/.bashrc
source ~/.bashrc

```

---

## 6. Plugin Gazebo ROS (roscam)

Langkah ini mengintegrasikan kamera yang dapat dideteksi sebagai *ROS topic*.

```bash
source /opt/ros/noetic/setup.bash
cd ~/
git clone https://github.com/Abstrakx/ardupilot_gazebo_krti.git
cd ardupilot_gazebo_krti
catkin init
cd src
catkin_create_pkg ardupilot_gazebo
cd ..
catkin build

```

### Menghubungkan Model Kustom

Kita harus memastikan *plugin path* merujuk pada direktori ROS Noetic:

```bash
export GAZEBO_MODEL_PATH=~/ardupilot_gazebo_krti/src/ardupilot_gazebo/models:$GAZEBO_MODEL_PATH
export GAZEBO_PLUGIN_PATH=/opt/ros/noetic/lib:$GAZEBO_PLUGIN_PATH

```

---

## 7. Menjalankan Simulasi

Untuk menjalankan simulasi secara utuh, kita perlu membuka beberapa jendela terminal:

### Terminal 1: Menjalankan Gazebo

```bash
source /opt/ros/noetic/setup.bash
source ~/ardupilot_gazebo_krti/devel/setup.bash
roslaunch ardupilot_gazebo krti.launch

```

### Terminal 2: Menjalankan SITL ArduPilot

```bash
cd ~/ardupilot/ArduCopter
sim_vehicle.py -f gazebo-iris --console --map

```

### Terminal 3: Menjalankan MAVROS

```bash
roslaunch mavros apm.launch fcu_url:=udp://:14550@

```

### Terminal 4: Visualisasi Kamera

Kita gunakan `rqt` untuk melihat *image feed* secara *real-time*:

```bash
rqt

```

Di dalam jendela **RQT**, silakan kita pilih menu:
`Plugins -> Visualization -> Image View`
Kemudian pilih *topic*: `/roscam/cam/image_raw`

---
