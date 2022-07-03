<template>
  <div class="box">
    <el-button @click="dialogFrom = true">新增</el-button>
    <el-dialog :visible.sync="dialogFrom" title="新增教学场景" width="600px">
      <div class="modal-body">
        <!-- 步骤 -->
        <div class="wizard" style="margin-bottom:20px;">
          <ul class="setps" style="margin-left:0px">
            <li :class="{ 'active': flagView === 1 }">
              <span class="badge">1</span>
              <span>基本配置</span>
            </li>
            <li :class="{ 'active': flagView === 2 }">
              <span class="badge">2</span>
              <span>基本配置</span>
            </li>
            <li :class="{ 'active': flagView === 3 }">
              <span class="badge">3</span>
              <span>基本配置</span>
            </li>
          </ul>
        </div>
        <!-- 基本信息 -->
        <el-form :model="form" :rules="rules" ref="form" v-show="flagView === 1">
          <!-- 场景名 -->
          <el-form-item label="场景名" label-width="100px" required prop="secenName">
            <el-input type="text" v-model="form.secenName" placeholder="2 ~ 20位字符" @change="validateFlag('form')">
            </el-input>
          </el-form-item>
          <!-- 虚拟化类型，教室 -->
          <div class="form-group">
            <el-form-item label="虚拟化类型" label-width="100px" prop="resourceType">
              <el-select v-model="form.resourceType">
                <el-option label="KVM" value="KVM"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="教室" label-width="100px">
              <el-select v-model="chClass">
                <el-option v-for="item in classs" :key="item" :label="item" :value="item"></el-option>
              </el-select>
            </el-form-item>
          </div>
          <!-- 网络 -->
          <div class="ip">
            <span class="iplabel">IPv4子网</span>
            <span>{{ ipv4 }}</span>
          </div>
          <div class="ip">
            <span class="iplabel">IPv6子网</span>
            <span>{{ ipv6 }}</span>
          </div>
          <div class="ip">
            <span class="iplabel">裸机环境</span>
            <el-switch v-model="rawEnv" active-color="#2b80c0" inactive-color="#959595">
            </el-switch>
            <el-popover placement="bottom" title="" width="300" trigger="hover"
              content="1、开启后，可创建安装操作系统的场景桌面；2、桌面安装操作系统，会使服务器IO占用过高，影响其他场景桌面使用。3、裸机环境的桌面不支持替换到新节点。">
              <!-- <ul>
                <li>1、开启后，可创建安装操作系统的场景桌面；</li>
                <li>2、桌面安装操作系统，会使服务器IO占用过高，影响其他场景桌面使用。</li>
                <li>3、裸机环境的桌面不支持替换到新节点。</li>
              </ul> -->
              <li class="tip" slot="reference">?</li>
            </el-popover>
          </div>
          <hr v-show="!rawEnv" />
          <div class="clearfix" v-show="!rawEnv">
            <div style="margin-top: 10px">请选择一个教学模板</div>
            <div class="clearfix-header">
              <ul>
                <li :class="{ 'active': 'Windows' === imageType }" @click="changDate('Windows')">Windows</li>
                <li :class="{ 'active': 'Linux' === imageType }" @click="changDate('Linux')">Linux</li>
                <li :class="{ 'active': 'other' === imageType }" @click="changDate('other')">其它</li>
              </ul>
            </div>
            <el-form-item class="clearfix-body">
              <el-table :data="tableData" @row-click="getRow">
                <el-table-column label="选择" width="70">
                  <template slot-scope="scope">
                    <el-radio v-model="form.radioRespon" :label="scope.row.modalName"> </el-radio>
                  </template>
                </el-table-column>
                <el-table-column prop="modalName" label="模板名">
                </el-table-column>
                <el-table-column prop="system" label="操作系统">
                </el-table-column>
                <el-table-column prop="Owner" label="属主" width="70">
                </el-table-column>
              </el-table>
            </el-form-item>
          </div>
          <!--裸机环境 -->
          <div class="ip" v-show="rawEnv">
            <span class="iplabel">固件</span>
            <div>
              <el-radio v-model="radio" label="BIOS"></el-radio>
              <el-radio v-model="radio" label="UEFI"></el-radio>
              <el-popover placement="bottom" title="" width="250" trigger="hover" content="UEFI模式下仅支持64位操作系统。">
                <li class="tip" slot="reference">?</li>
              </el-popover>
            </div>
          </div>
          <el-form-item label="安装包" label-width="100px" v-show="rawEnv" required>
            <el-select v-model="chIos" class="iosSelect">
              <el-option v-for="item in ioss" :key="item.name" :label="item.name" :value="item.name"></el-option>
            </el-select>
          </el-form-item>
          <div class="prompt" v-show="rawEnv">* 裸机系统安装后，请先安装Guesttool再使用桌面</div>
          <div class="ip" v-show="rawEnv">
            <span class="iplabel">系统类型</span>
            <div class="">
              {{ system }}
            </div>
          </div>
          <div class="text-right">
            <button type="button" class="btn btn-sm btn-default" :class="{ 'btn-choice': firstFlag }"
              :disabled="!firstFlag" @click="logError('form')">
              下一步
              <i class="el-icon-right"></i>
            </button>
          </div>
        </el-form>
        <el-form :model="secdfrom" ref="secdfrom" :rules="rules" v-show="flagView === 2">
          <!-- 硬件配置 -->
          <el-form-item label="硬件配置" label-width="100px" class="el-hardwares" prop="hardwareId">
            <el-select v-model="secdfrom.hardwareId" class="hardwareSelect" @change="validateFlag('secdfrom')">
              <el-option v-for="item in hardwares" :key="item.name" :label="item.name" :value="item.id">
              </el-option>
            </el-select>
            <el-popover placement="bottom" title="" width="250" trigger="hover"
              content="此处筛选出符合以下要求的硬件配置:1、系统盘的容量大于等于所选教学模板的系统盘;2、数据盘的个数需要大于等于所选教学模板数据盘的个数;3、数据盘的容量要大于等于所选教学模板数据盘的容量，如果有两块数据盘，则依次比较。创建出的教学桌面或裸机环境桌面的配置:1、所创建桌面的CPU和内存与硬件配置保持一致;2、以教学模板所创建教学桌面系统盘数据盘大小与教学模板保持一致，新增数据盘以硬件配置数据盘大小保持一致。">
              <li class="tip" slot="reference">?</li>
            </el-popover>
          </el-form-item>
          <div class="hardwares">
            <span>CPU {{ hardwares[secdfrom.hardwareId].cpu }}核</span>
            <span>内存 {{ hardwares[secdfrom.hardwareId].Memory }}G</span>
            <span>系统盘 {{ hardwares[secdfrom.hardwareId].systemDisk }}G</span>
            <span>数据盘 {{ hardwares[secdfrom.hardwareId].systemDisk }}G</span>
          </div>
          <!-- usb -->
          <el-form-item label="USB端口" label-width="100px">
            <el-select v-model="usb" class="el-usbSelect">
              <el-option laber="不使用" value="不使用"></el-option>
              <el-option laber="2.0重定向" value="2.0重定向"></el-option>
              <el-option laber="3.0重定向" value="3.0重定向"></el-option>
            </el-select>
            <el-popover placement="bottom" title="" width="250" trigger="hover"
              content="1.使用USB 3.0重定向，则在桌面内采用USB 3.0的控制器；2.使用USB 2.0重定向，则在桌面内采用USB 2.0的控制器；3.无论启用何种方式，任意协议类型的USB设备可以接入终端任意协议类型的接口上重定向使用；4.KVM虚拟化类型下，Windows 7及以上操作系统推荐使用USB 3.0重定向，Windows XP系统只能使用USB 2.0重定向。">
              <li class="tip" slot="reference">?</li>
            </el-popover>
            <el-checkbox v-model="show_desk_info" class="el-show_desk_info">屏幕水印</el-checkbox>
            <el-popover placement="bottom" title="" width="250" trigger="hover"
              content="1. 使用加速（不安全）选项，是指在保障磁盘一定读速度的前提下，实现磁盘写加速。但当遇到服务器异常关机或重启时，会有一定机率造成数据丢失。">
              <li class="tip" slot="reference">?</li>
            </el-popover>
          </el-form-item>
          <!-- 磁盘缓存 -->
          <el-form-item label="磁盘缓存" label-width="100px">
            <el-select v-model="disk_cachemode" class="el-cacheSelect">
              <el-option laber="不使用" value="不使用"></el-option>
              <el-option laber="加速(不安全)" value="加速"></el-option>
            </el-select>
            <el-popover placement="bottom" title="" width="250" trigger="hover"
              content="1、启用半虚拟化声卡后， 桌面内视频播放更加流畅，声音效果更佳；</br>2、启用后，桌面无法执行挂起、迁移操作。">
              <li class="tip" slot="reference">?</li>
            </el-popover>
            <el-checkbox v-model="half_virtual_sound_card" class="el-show_desk_info">半虚拟化声卡</el-checkbox>
            <el-popover placement="bottom" title="" width="250" trigger="hover"
              content="1、启用半虚拟化声卡后， 桌面内视频播放更加流畅，声音效果更佳；</br>2、启用后，桌面无法执行挂起、迁移操作。">
              <li class="tip" slot="reference">?</li>
            </el-popover>
          </el-form-item>
          <!-- 系统盘还原 -->
          <el-form-item label="系统盘还原" label-width="100px">
            <el-select v-model="rollback" class="el-cacheSelect">
              <el-option laber="不还原" value="不还原"></el-option>
              <el-option laber="每次还原" value="每次还原"></el-option>
              <el-option laber="每天还原" value="每天还原"></el-option>
              <el-option laber="每周还原" value="每周还原"></el-option>
              <el-option laber="每月还原" value="每月还原"></el-option>
            </el-select>
            <el-select v-model="rollback2" class="el-back" v-show="rollback === '每次还原'">
              <el-option laber="开机还原" value="开机还原"></el-option>
              <el-option laber="开机+重启还原" value="开机+重启还原"></el-option>
              <el-option laber="关机还原" value="关机还原"></el-option>
            </el-select>
            <el-select v-model="rollback_weekday" class="el-back" v-show="rollback === '每周还原'">
              <el-option v-for="item in weeks" :key="item" :label="item" :value="item"></el-option>
            </el-select>
            <el-select v-model="rollback_monthday" class="el-back" v-show="rollback === '每月还原'">
              <el-option v-for="item in monthday" :key="item" :label="item" :value="item"></el-option>
            </el-select>
          </el-form-item>
          <!-- 数据盘还原 -->
          <el-form-item label="数据盘还原" label-width="100px">
            <el-select v-model="data_rollback" class="el-cacheSelect">
              <el-option laber="不还原" value="不还原"></el-option>
              <el-option laber="每次还原" value="每次还原"></el-option>
              <el-option laber="每天还原" value="每天还原"></el-option>
              <el-option laber="每周还原" value="每周还原"></el-option>
              <el-option laber="每月还原" value="每月还原"></el-option>
            </el-select>
            <el-select v-model="data_rollback2" class="el-back" v-show="data_rollback === '每次还原'">
              <el-option laber="开机还原" value="开机还原"></el-option>
              <el-option laber="开机+重启还原" value="开机+重启还原"></el-option>
              <el-option laber="关机还原" value="关机还原"></el-option>
            </el-select>
            <el-select v-model="data_rollback_weekday" class="el-back" v-show="data_rollback === '每周还原'">
              <el-option v-for="item in weeks" :key="item" :label="item" :value="item"></el-option>
            </el-select>
            <el-select v-model="data_rollback_weekday" class="el-back" v-show="data_rollback === '每月还原'">
              <el-option v-for="item in monthday" :key="item" :label="item" :value="item"></el-option>
            </el-select>
          </el-form-item>
          <!-- 计算机名 -->
          <el-form-item label="计算机名" label-width="100px">
            <div class="el-name">
              <el-select v-model="hostNameType" class="el-NameType">
                <el-option laber="前缀 + 1位数字" value="前缀 + 1位数字"></el-option>
                <el-option laber="前缀 + 2位数字" value="前缀 + 2位数字"></el-option>
                <el-option laber="前缀 + 3位数字" value="前缀 + 3位数字"></el-option>
              </el-select>
              <el-form-item prop="hostNamePre" class="el-NamePre">
                <el-input type="text" @change="validateFlag('secdfrom')" v-model="secdfrom.hostNamePre"
                  placeholder="前缀，2 ~ 12位字符">
                </el-input>
              </el-form-item>
              <el-form-item prop="hostNameBegin">
                <el-input-number v-model="secdfrom.hostNameBegin" controls-position="right" :min="1" :max="maxNumHost"
                  class="el-NameBegin" @change="validateFlag('secdfrom')">
                </el-input-number>
              </el-form-item>
            </div>
          </el-form-item>
          <!-- 用户名 -->
          <el-form-item label="用户名" label-width="100px">
            <div class="el-name">
              <el-select v-model="userNameType" class="el-NameType">
                <el-option laber="不使用" value="不使用"></el-option>
                <el-option laber="前缀 + 1位数字" value="前缀 + 1位数字"></el-option>
                <el-option laber="前缀 + 2位数字" value="前缀 + 2位数字"></el-option>
                <el-option laber="前缀 + 3位数字" value="前缀 + 3位数字"></el-option>
              </el-select>
              <el-form-item class="el-NamePre" prop="userNamePre">
                <el-input type="text" v-model="secdfrom.userNamePre" placeholder="前缀，2 ~ 12位字符"
                  :disabled="userNameType === '不使用'" @change="validateFlag('secdfrom')">
                </el-input>
              </el-form-item>
              <el-form-item prop="userNameBegin">
                <el-input-number v-model="secdfrom.userNameBegin" controls-position="right" :min="1" :max="maxNumUser"
                  :disabled="userNameType === '不使用'" class="el-NameBegin" @change="validateFlag('secdfrom')">
                </el-input-number>
              </el-form-item>
            </div>
          </el-form-item>
          <div>
            注：使用用户名修改功能，桌面新建后会自动开启桌面完成修改并关闭桌面，请耐心等待。
          </div>
          <div class="text-right">
            <button type="button" class="btn btn-sm btn-default firstbtn btn-choice " @click="next(1)">
              <i class="el-icon-back"></i>
              上一步
            </button>
            <button type="button" class="btn btn-sm btn-default" :class="{ 'btn-choice': secondFlag }"
              :disabled="!secondFlag" @click="logError('secdfrom')">
              下一步
              <i class="el-icon-right"></i>
            </button>
          </div>
        </el-form>
        <el-form :rules="rules" v-show="flagView === 3" :model="hostfrom" ref="hostfrom">
          <el-form-item label="创建方式" label-width="100px">
            <el-select v-model="hostType">
              <el-option label="均衡创建" value="0"></el-option>
              <el-option label="自定义" value="1"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="资源池" label-width="100px">
            <el-select v-model="hostfrom.resourceId" style="width: 460px"
              @change="resourceIdChange(hostfrom.resourceId)">
              <el-option v-for="item of resourceList" :label="item.resourceName" :key="item.resourceId"
                :value="item.resourceId"></el-option>
            </el-select>
          </el-form-item>
          <!-- 计算节点 -->
          <el-form-item label="计算节点" label-width="100px" required>
            <div class="tagarea">
              <el-checkbox-group v-model="checkboxGroup" @change="handleCheckedHostChange" v-show="hostType === '0'">
                <el-checkbox-button v-for="item in hostdata" :label="item.id" :key="item.id">{{ item.name }}
                </el-checkbox-button>
              </el-checkbox-group>
              <div v-show="hostType === '1'" v-for="(item, index) of hostdata" :key="item.id">
                <span>{{ item.name }}</span>
                <el-input-number v-model="hostdata[index].input" controls-position="right" :min="0" :max="item.maxData"
                  :disabled="false" class="el-hostNum" @change="changeArrHost">
                </el-input-number>
                <p class="text" v-show="hostType === '1'">最大{{ item.maxData }}</p>
              </div>
            </div>
            <div class="checkbnt">
              <el-checkbox class="el-checkboxGroup" v-model="checkAll" @change="handleCheckAllChange"
                v-show="hostType === '0'">全选</el-checkbox>
              <button type="button" class="btn btn-sm btn-default firstbtn text-left"
                :disabled="checkboxGroup.length === 0" :class="{ 'btn-choice': checkboxGroup.length !== 0 }"
                @click="twoDialog">
                高级
              </button>
            </div>
          </el-form-item>
          <!-- 桌面数 -->
          <el-form-item label="桌面数" label-width="100px" prop="hostNum">
            <el-input-number v-model="hostfrom.hostNum" controls-position="right" :min="0" :max="30"
              :disabled="checkboxGroup.length === 0" class="el-hostNum" v-show="hostType === '0'" @change="checkFlag">
            </el-input-number>
            <span class="text" v-show="hostType === '0'">最大30</span>
            <span v-show="hostType === '1'" class="textNum">{{ hostfrom.hostNum }}</span>
          </el-form-item>
          <div class="text-right">
            <button type="button" class="btn btn-sm btn-default firstbtn btn-choice " @click="next(2)">
              <i class="el-icon-back"></i>
              上一步
            </button>
            <button type="button" class="btn btn-sm btn-default" :class="{ 'btn-choice': thirdFlag }"
              :disabled="!thirdFlag" @click="logError('ok')">
              完成
              <i class="el-icon-check"></i>
            </button>
          </div>
        </el-form>
      </div>
    </el-dialog>
    <el-dialog :visible.sync="dialogtable" title="高级" width="600px" class="dialogtable">
      <el-form :model="hostListFrom" :rules="rules" ref="hostListFrom">
        <el-form-item label="主机" label-width="160px">
          <el-select v-model="selectResourceId" class="el-resourceId">
            <el-option v-for="item of hostConfig" :label="item.name" :key="item.id" :value="item.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="系统盘镜像存储设备" label-width="160px" prop="image_storage_performance">
          <el-select v-model="hostListFrom.image_storage_performance" class="el-resourceId" @change="validateFlag('hostListFrom')">
            <el-option v-for="item of hostConfig[selectResourceId].image_storage_performance" :label="item.name"
              :key="item.image_storage_performance_id" :value="item.image_storage_performance_id"></el-option>
          </el-select>
          <el-progress :percentage="setNum(429, 492)" class="el-progress"></el-progress>
          <span>429/492GB</span>
        </el-form-item>
        <el-form-item label="数据盘镜像存储设备" label-width="160px" prop="image_storage_capacity">
          <el-select v-model="hostListFrom.image_storage_capacity" class="el-resourceId" @change="validateFlag('hostListFrom')">
            <el-option v-for="item of hostConfig[selectResourceId].image_storage_capacity" :label="item.name"
              :key="item.image_storage_capacity_id" :value="item.image_storage_capacity_id"></el-option>
          </el-select>
          <el-progress :percentage="setNum(256, 492)" class="el-progress"></el-progress>
          <span>256/492GB</span>
        </el-form-item>
        <el-form-item label="系统盘存储设备" label-width="160px" prop="storage_performance">
          <el-select v-model="hostListFrom.storage_performance" class="el-resourceId"  @change="validateFlag('hostListFrom')">
            <el-option v-for="item of hostConfig[selectResourceId].storage_performance" :label="item.name"
              :key="item.storage_performance_id" :value="item.storage_performance_id"></el-option>
          </el-select>
          <el-progress :percentage="setNum(351, 492)" class="el-progress"></el-progress>
          <span>351/492GB</span>
        </el-form-item>
        <el-form-item label="数据盘存储设备" label-width="160px" prop="storage_capacity">
          <el-select v-model="hostListFrom.storage_capacity" class="el-resourceId"  @change="validateFlag('hostListFrom')">
            <el-option v-for="item of hostConfig[selectResourceId].storage_capacity" :label="item.name"
              :key="item.storage_capacity_id" :value="item.storage_capacity_id"></el-option>
          </el-select>
          <el-progress :percentage="setNum(256, 492)" class="el-progress"></el-progress>
          <span>256/492GB</span>
        </el-form-item>
      </el-form>
      <div class="text-right">
        <button type="button" class="btn btn-sm btn-default firstbtn" :class="{'btn-choice':fourFlag}"  @click="dialogtable = false" :disabled="!fourFlag">
          确认
        </button>
        <button type="button" class="btn btn-sm btn-default firstbtn btn-choice"
          @click="dialogtable = false">
          取消
        </button>
        <button type="button" class="btn btn-sm btn-default" :class="{'btn-choice':fourFlag}" :disabled="!fourFlag"
          @click="logError('ok')">
          应用
        </button>
      </div>
    </el-dialog>
  </div>
</template>

<script>

export default {
  name: 'HelloWorld',
  data() {
    const checkHostNum = (rule, value, callback) => {
      if (value === 0) {
        return callback(new Error(''));
      }
      callback()
    }
    return {
      dialogFrom: false,
      rawEnv: false,
      flagView: 1,
      firstFlag: false,
      secondFlag: false,
      thirdFlag: false,
      fourFlag: false,
      dialogtable: false,
      resourceType: 'KVM',  // 虚拟化类型
      hardware: 'rh-test2', // 硬件配置
      resourceTypelist: [],
      radio: 'BIOS',
      chClass: 'aaa',
      usb: '不使用',
      chIos: {
        name: 'uniontechos-desktop-20-professional-1050-amd64.iso',
        system: 'uniont'
      },
      hardwareId: 0, // del
      secdfrom: {
        hardwareId: 0,
        hostNamePre: 'PC',
        userNamePre: 'User',
        hostNameBegin: 1,
        userNameBegin: 1
      },
      hardwares: [{
        name: 'rh-test2',
        cpu: '16',
        Memory: '24',
        systemDisk: '200',
        id: 0
      }, {
        name: 'htrh',
        cpu: '16',
        Memory: '24',
        systemDisk: '100',
        dateDisk: '15',
        id: 1
      }],
      system: 'uniont',  // 系统类型
      show_desk_info: false,
      disk_cachemode: '不使用',
      half_virtual_sound_card: false,
      rollback: '不还原',
      data_rollback: '不还原',
      rollback2: '',
      rollback_weekday: '',
      rollback_monthday: 1,
      data_rollback2: '',
      data_rollback_weekday: '',
      data_rollback_monthday: 1,
      weeks: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
      monthday: [],
      hostNamePre: 'PC',
      hostNameBegin: 1,
      hostNameType: '前缀 + 1位数字',
      userNameType: '不使用',
      userNamePre: 'User',
      userNameBegin: 1,
      maxNumHost: 9,
      maxNumUser: 9,
      classs: [
        'aaa',
        'default',
        'vdi'
      ],
      form: {
        secenName: '',
        resourceType: 'KVM',  // 虚拟化类型
        radioRespon: ''
      },
      rules: {
        secenName: [
          { required: true, trigger: 'blur' },
          { min: 2, max: 20, message: '长度在 2 到 20 个字符', trigger: 'blur' }
        ],
        hostNum: [
          { required: true, trigger: 'change' },
          { validator: checkHostNum, trigger: 'change' }
        ],
        hardwareId: [
          { required: true, trigger: 'blur' }
        ],
        hostNamePre: [
          { required: true, trigger: 'blur' },
          { min: 2, max: 12, trigger: 'change' }
        ],
        userNamePre: [
          { required: true, trigger: 'blur' },
          { min: 2, max: 12, trigger: 'change' }
        ],
        hostNameBegin: [
          { required: true, trigger: 'change' }
        ],
        userNameBegin: [
          { required: true, trigger: 'change' }
        ],
        image_storage_performance: [
          { required: true, trigger: 'blur' }
        ],
        image_storage_capacity: [
          { required: true, trigger: 'blur' }
        ],
        storage_performance: [
          { required: true, trigger: 'blur' }
        ],
        storage_capacity: [
          { required: true, trigger: 'blur' }
        ]
      },
      imageType: 'Windows',
      radioRespon: '',
      ipv4: '',
      ipv6: '',
      ipList: [{
        className: 'aaa',
        ipv4: '123(172.16.92.100~172.16.92.110)',
        ipv6: ''
      }, {
        className: 'default',
        ipv4: 'default(172.16.2.10~172.16.21.115)',
        ipv6: ''
      }, {
        className: 'vdi',
        ipv4: '',
        ipv6: 'dddd(fefe::1~fefe::1000)'
      }],
      tableDataList: [{
        class: 'aaa',
        modalName: 'ngn',
        system: 'Windows 7 (64 bit)',
        Owner: '23',
        imageType: 'Windows'
      }, {
        class: 'aaa',
        modalName: 'nsrt',
        system: 'Windows 10 (64 bit)',
        Owner: '23',
        imageType: 'Windows'
      }, {
        class: 'default',
        modalName: 'jyt',
        system: 'Windows 7 (64 bit)',
        Owner: '23',
        imageType: 'Windows'
      }, {
        class: 'default',
        modalName: '73',
        system: 'Windows 7 (64 bit)',
        Owner: '23',
        imageType: 'Linux'
      }, {
        class: 'default',
        modalName: 'ngn25',
        system: 'ubuntu',
        Owner: '23',
        imageType: 'Linux'
      }, {
        class: 'vdi',
        modalName: ',mygh',
        system: 'Windows 7 (64 bit)',
        Owner: '23',
        imageType: 'Windows'
      }, {
        class: 'default',
        modalName: '375',
        system: 'Windows 10 (64 bit)',
        Owner: '23',
        imageType: 'Windows'
      }, {
        class: 'vdi',
        modalName: 'ngdd',
        system: 'Windows 7 (64 bit)',
        Owner: '23',
        imageType: 'Windows'
      }, {
        class: 'default',
        modalName: '37ntg',
        system: 'Windows 10 (64 bit)',
        Owner: '23',
        imageType: 'Windows'
      }, {
        class: 'vdi',
        modalName: '73253',
        system: 'Windows 7 (64 bit)',
        Owner: '23',
        imageType: 'Linux'
      }, {
        class: 'default',
        modalName: '7ng3',
        system: 'Windows 7 (64 bit)',
        Owner: '23',
        imageType: 'Linux'
      }, {
        class: 'vdi',
        modalName: '3773',
        system: 'Windows 7 (64 bit)',
        Owner: '23',
        imageType: 'Linux'
      }],
      tableData: [],
      ioss: [{
        name: 'uniontechos-desktop-20-professional-1050-amd64.iso',
        system: 'uniont'
      }, {
        name: 'cn_windows_7_ultimate_with_sp1_x86_dvd_u_677486.iso',
        system: 'Windows'
      }, {
        name: 'OS-Easy-CloudDesktop-Server-5.4.0.iso',
        system: 'Linux'
      }, {
        name: 'Kylin-Desktop-V10-SP1-Release-2107-x86_64.iso',
        system: 'Linux'
      }],

      // 3
      hostType: '0', // 创建方式
      resourceId: 0, // 资源池
      resourceList: [{
        resourceName: 'default',
        resourceId: 0
      }, {
        resourceName: '666',
        resourceId: 1
      }],
      hostfrom: {
        resourceId: 0,
        hostNum: 0,
      },
      hostConfig: [{
        id: 0,
        name: 'vdi-12df0f',
        image_storage_performance: [{
          image_storage_performance_id: 0,
          name: 'local_ssd',
          num1: 429,
          num2: 500
        }],
        image_storage_capacity: [{
          image_storage_capacity_id: 0,
          name: 'local_ssd',
          num1: 429,
          num2: 500
        }, {
          image_storage_capacity_id: 1,
          name: 'lun86',
          num1: 236,
          num2: 492
        }, {
          image_storage_capacity_id: 2,
          name: 'getda',
          num1: 396,
          num2: 492
        }],
        storage_performance: [{
          storage_performance_id: 0,
          name: 'local_stat',
          num1: 429,
          num2: 500
        }],
        storage_capacity: [{
          storage_capacity_id: 0,
          name: 'local_ssd',
          num1: 429,
          num2: 500
        }]
      },
      {
        id: 1,
        name: 'vdi-12325',
        image_storage_performance: [{
          image_storage_performance_id: 0,
          name: 'local_ssd',
          num1: 429,
          num2: 500
        }],
        image_storage_capacity: [{
          image_storage_capacity_id: 0,
          name: 'local_ssd',
          num1: 429,
          num2: 500
        }, {
          image_storage_capacity_id: 1,
          name: 'bdf',
          num1: 236,
          num2: 492
        }, {
          image_storage_capacity_id: 2,
          name: 'getda',
          num1: 396,
          num2: 492
        }],
        storage_performance: [{
          storage_performance_id: 0,
          name: 'local_stat',
          num1: 429,
          num2: 500
        }],
        storage_capacity: [{
          storage_capacity_id: 0,
          name: 'local_ssd',
          num1: 429,
          num2: 500
        }]
      },
      {
        id: 2,
        name: 'vdi-1iik0f',
        image_storage_performance: [{
          image_storage_performance_id: 0,
          name: 'local_ssd',
          num1: 429,
          num2: 500
        }],
        image_storage_capacity: [{
          image_storage_capacity_id: 0,
          name: 'local_ssd',
          num1: 429,
          num2: 500
        }, {
          image_storage_capacity_id: 1,
          name: 'lun86',
          num1: 236,
          num2: 492
        }],
        storage_performance: [{
          storage_performance_id: 0,
          name: 'local_stat',
          num1: 429,
          num2: 500
        }],
        storage_capacity: [{
          storage_capacity_id: 0,
          name: 'local_ssd',
          num1: 429,
          num2: 500
        }]
      }
      ],
      hosts: [{
        id: 0,
        resourceId: 0,
        name: 'vdi-12df0f',
        maxData: 20,
      }, {
        id: 1,
        resourceId: 0,
        name: 'vdi-12325',
        maxData: 10,
      }, {
        id: 2,
        resourceId: 1,
        name: 'vdi-1iik0f',
        maxData: 20,
      }],
      hostdata: [{
        id: 0,
        resourceId: 0,
        name: 'vdi-12df0f',
        maxData: 20,
        input: null
      }, {
        id: 1,
        resourceId: 0,
        name: 'vdi-12325',
        maxData: 10,
        input: null
      }],
      toTwoDialog: [],
      twoDialogList: [],
      selectHost: null,
      hostListFrom: {
        image_storage_performance: 0,
        image_storage_capacity: 0,
        storage_performance: 0,
        storage_capacity: 0
      },
      checkboxGroup: [],
      checkAll: false,
      arrHostNum: [],
      hostNum: 0,
      selectResourceId: 0,
    }
  },
  watch: {
    chIos: function (val) {
      switch (val) {
        case 'uniontechos-desktop-20-professional-1050-amd64.iso': this.system = 'uniont'; break;
        case 'cn_windows_7_ultimate_with_sp1_x86_dvd_u_677486.iso': this.system = 'Windows'; break;
        case 'OS-Easy-CloudDesktop-Server-5.4.0.iso': this.system = 'Linux'; break;
        case 'Kylin-Desktop-V10-SP1-Release-2107-x86_64.iso': this.system = 'Linux'; break;
        default: this.system = '';
      }
    },
    hostNameType: function (val) {
      switch (val) {
        case '前缀 + 1位数字': this.maxNumHost = 9; break;
        case '前缀 + 2位数字': this.maxNumHost = 99; break;
        case '前缀 + 3位数字': this.maxNumHost = 999; break;
        default: break;
      }
    },
    userNameType: function (val) {
      switch (val) {
        case '不使用': this.secdfrom.userNamePre = 'User', this.secdfrom.userNameBegin = 1; break;
        case '前缀 + 1位数字': this.maxNumUser = 9; break;
        case '前缀 + 2位数字': this.maxNumUser = 99; break;
        case '前缀 + 3位数字': this.maxNumUser = 999; break;
        default: break;
      }
    },
    chClass: function (val) {
      this.getdata(val);
      this.getIp(val);
    },
    hostType: function () {   // 资源池更改
      this.checkboxGroup = [];
      this.checkAll = false;
    }
  },
  methods: {
    getRow: function (row) {
      this.form.radioRespon = row.modalName;
    },
    resourceIdChange: function (val) {
      let arr = [];
      this.hosts.forEach(item => {
        if (item.resourceId === val) {
          item.input = null;
          arr.push(item);
        }
      })
      this.hostdata = arr;
      this.checkboxGroup = [];
      this.checkAll = false;
    },
    changDate: function (val) {
      if (this.imageType !== val) {
        this.imageType = val;
        this.tableData = this.tableDataList.filter((item) => {
          return item.imageType === val;
        })
      }
    },
    getdata: function (val) {
      this.tableData = this.tableDataList.filter((item) => {
        return item.imageType === 'Windows' && item.class === val;
      })
      this.form.radioRespon = this.tableData[0].modalName;
    },
    getIp: function (val) {
      this.ipList.forEach(item => {
        if (item.className === val) {
          this.ipv4 = item.ipv4;
          this.ipv6 = item.ipv6;
        }
      });
    },
    validateFlag: function (val) {   // 验证判断
      let flag = true;
      this.$refs[val].validate((valid) => {
        if (!valid) {
          flag = false;
        }
      })
      if (flag) {
        if (val === 'form') {
          this.firstFlag = true;
        } else if (val === 'secdfrom') {
          this.secondFlag = true;
        } else if (val === 'hostListFrom') {
          this.fourFlag = true;
        } else {
          this.thirdFlag = true;
        }
      } else {
        if (val === 'form') {
          this.firstFlag = false;
        } else if (val === 'secdfrom') {
          this.secondFlag = false;
        } else if (val === 'hostListFrom') {
          this.fourFlag = false;
        } else {
          this.thirdFlag = false;
        }
      }
    },
    logError: function (val) {
      if (val === 'form') {
        this.flagView = 2;
      } else if (val === 'secdfrom') {
        this.flagView = 3;
      } else {
        console.log('ok');
      }
    },
    setMonthday: function () {
      for (let i = 0; i < 29; ++i) {
        this.monthday.push(i);
      }
    },
    next: function (val) {
      this.flagView = val;
    },
    handleCheckAllChange: function () {
      if (this.hostdata.length !== this.checkboxGroup.length) {
        this.hostdata.forEach(item => {
          if (this.checkboxGroup.indexOf(item.id) === -1) {
            this.checkboxGroup.push(item.id)
          }
        })
      } else {
        this.checkboxGroup = [];
      }
    },
    handleCheckedHostChange: function () {
      this.checkAll = this.hostdata.length === this.checkboxGroup.length
      if (this.checkboxGroup.length === 0) {
        this.thirdFlag = false;
      }
    },
    changeArrHost: function () {
      let num = 0;
      this.checkboxGroup = [];
      this.hostdata.forEach(item => {
        if (item.input && item.input !== 0) {
          num += item.input;
          this.checkboxGroup.push(item.id)
        }
      })
      this.hostfrom.hostNum = num;
      if (num > 0) {
        this.thirdFlag = true;
      } else {
        this.thirdFlag = false;
      }
    },
    checkFlag: function () {
      if (this.hostfrom.hostNum > 0) {
        this.thirdFlag = true;
      } else {
        this.thirdFlag = false;
      }
    },
    setNum: function (val1, val2) {
      return parseInt((val1 / val2).toFixed(1) * 100)
    },
    twoDialog: function () {
      let arr = [];
      this.dialogtable = true;
      if (this.hostType === '1') {
        this.hostdata.forEach(item => {
          if (item.input !== 0) {
            arr.push(item.id);
          }
        })
      } else {
        arr = this.checkboxGroup;
      }
      arr.forEach(item => {
        this.hostConfig.forEach(it => {
          if (it.id === item) {
            this.toTwoDialog.push(it);
          }
        })
      })
      this.selectHost = this.toTwoDialog[0].id;
    }
  },
  mounted: function () {
    this.getdata(this.chClass);
    this.setMonthday();
    this.getIp(this.chClass)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
::v-deep {
  .el-dialog__header {
    border-bottom: 1px #eee solid;
  }

  .el-dialog__body {
    padding: 20px 20px;
    color: #606266;
    font-size: 14px;
    word-break: break-all;
  }

  .el-form-item__label {
    &::before {
      position: absolute;
      left: 90px;
    }
  }

  .el-form-item__error {
    display: none;
  }

  .el-form-item.is-error .el-input__inner {
    background-color: #fff0f0;
  }

  .el-input__inner {
    height: 40px;
  }

  .el-input__icon {
    line-height: 32px;
  }

  .el-input-number__increase {
    height: 14px;
    line-height: 14px;
    width: 20px;
    top: 3px;
  }

  .el-table__row .el-radio__label {
    display: none;
  }

  .el-input-number.is-controls-right .el-input-number__decrease {
    bottom: 0px;
    width: 20px;
    height: 14px;
    line-height: 14px;
    bottom: 3px;
  }

  .el-table th.el-table__cell {
    background-color: #ddd;
    padding: 5.5px 0px;
  }

  .el-table__row {
    padding: 0px;
  }

  .el-table .el-table__cell {
    padding: 2px 0px;
  }

  .el-radio__input.is-checked+.el-radio__label {
    font-weight: 700;
    color: #515A6E;
  }

  .el-radio {
    margin-right: 15px;
  }

  .el-checkbox-button__inner {
    padding: 2px 5px;
    background-color: #ddd;
    border: 0px;
    margin: 0px 5px 5px 0px;
    border-radius: 4px;
    height: 22px;
    line-height: 22px;
  }

  .el-checkbox-button:first-child .el-checkbox-button__inner {
    border-radius: 4px;
  }

  .el-checkbox-button:last-child .el-checkbox-button__inner {
    border-radius: 4px;
  }

  .el-checkbox-button.is-checked .el-checkbox-button__inner {
    color: #FFF;
    background-color: #2b80c0;
    border-color: #409EFF;
    box-shadow: -1px 0 0 0 #8cc5ff;
  }

  .el-checkbox__input.is-checked+.el-checkbox__label {
    color: #515A6E;
    font-weight: 700;
  }

  .el-checkbox__inner {
    width: 18px;
    height: 18px;

    &::after {
      left: 6px;
      height: 9px;
    }
  }

  .el-form-item__content {
    line-height: 30px;
    position: relative;
    font-size: 14px;
  }
}

hr {
  margin-top: 18px;
  margin-bottom: 18px;
  border: 0;
  border-top: 1px solid #eee;
}

.modal-body {
  position: relative;
  padding-top: 0px;

  .wizard {
    overflow: hidden;
    background-color: #f9f9f9;
    border: 1px solid #dcdee2;
    border-radius: 4px;
    box-shadow: 0 1px 4px rgb(0 0 0 / 7%);

    ul {
      padding: 0;
      margin: 0;
      list-style: none outside none;

      li {
        position: relative;
        float: left;
        height: 46px;
        padding: 0 20px 0 30px;
        margin: 0;
        font-size: 16px;
        line-height: 46px;
        cursor: default;
        color: #515a6e;
        background-color: #e8eaec;

        // &::after {
        //   position: absolute;
        //   right: -14px;
        //   z-index: 1;
        //   content: '';
        //   width: 0;
        //   height: 0;
        //   border-top: 23px solid transparent;
        //   border-left: 14px solid;
        //   border-bottom: 23px solid transparent;
        // }
      }

      .active {
        color: #3a87ad;
        background: #e6f4ff;
        padding: 0 20px 0 20px;

        .badge {
          color: #fff;
          background-color: #2b80c0;
        }
      }

      .badge {
        display: inline-block;
        min-width: 10px;
        padding: 3px 7px;
        font-size: 12px;
        font-weight: 700;
        color: #fff;
        line-height: 1;
        vertical-align: baseline;
        white-space: nowrap;
        text-align: center;
        background-color: #999;
        border-radius: 10px;
        margin-right: 8px;
      }
    }
  }

  .form-group {
    display: flex;
  }

  .ip {
    display: flex;
    align-items: center;
    margin-bottom: 20px;

    .iplabel {
      text-align: right;
      width: 100px;
      padding-right: 13px;
      box-sizing: border-box;
    }

    span {
      display: inline-block;
    }
  }

  .tip {
    display: inline-block;
    width: 16px;
    height: 18px;
    color: #fff;
    background-color: #2b80c0;
    border-radius: 50%;
    text-align: center;
    line-height: 18px;
    margin-left: 10px;
  }

  .clearfix {
    margin: 0 0 30px;
    position: relative;
    border-radius: 0;
    padding: 0;

    .clearfix-header {
      height: 34px;
      color: #333;
      border: 1px solid #C2C2C2;
      background: #fafafa;
      margin-top: 10px;
      padding: 0;

      ul {
        display: flex;
        padding: 0px;
        margin: 0px;

        li {
          text-align: center;
          padding: 0px 15px;
          display: inline-block;
          line-height: 34px;
        }

        .active {
          font-weight: 700;
          background-color: #fff;
          border-top: 2px solid #2b80c0;
          border-left: 1px solid gray;
          border-right: 1px solid gray;
        }
      }
    }

    .clearfix-body {
      margin-top: 2px;
      height: 150px;
      overflow: auto;
      border: 1px solid gray;
    }
  }

  .iosSelect {
    width: 360px;
  }

  .prompt {
    margin-left: 100px;
    margin-bottom: 15px;
  }

  .text-left {
    position: absolute;
    right: 0px;
  }

  .hardwareSelect {
    width: 410px;
    margin-right: 15px;
  }

  .hardwares {
    margin-left: 100px;
    margin-top: 7px;
    margin-bottom: 20px;

    span {
      margin-right: 20px;
    }
  }

  .el-hardwares {
    margin-bottom: 0px;
  }

  .el-usbSelect {
    width: 120px;
    margin-right: 25px;
  }

  .el-show_desk_info {
    margin-left: 30px;
    margin-right: 20px;
  }

  .el-cacheSelect {
    width: 170px;
    margin-right: 20px;
  }

  .el-back {
    width: 120px;
    margin-left: 10px;
  }

  .el-name {
    display: flex;

    .el-NameType {
      width: 170px;
      margin-right: 30px;
    }

    .el-NamePre {
      width: 120px;
      margin-right: 20px;
    }

    .el-NameBegin {
      width: 120px;
    }
  }

  .tagarea {
    width: 460px;
    height: 120px;
    border: 1px rgba(128, 128, 128, 0.5) solid;
    padding: 10px;
    box-sizing: border-box;
    overflow-y: auto;

    span {
      display: inline-block;
      width: 200px;
      text-align: left;
    }

    .el-hostNum {
      width: 120px;
    }
  }

  .checkbnt {
    width: 460px;
    height: 30px;
    margin-bottom: 5px;
    position: relative;
  }

  .el-checkboxGroup {
    padding: 0px;
  }

  .text-left {
    margin-top: 5px;
    padding: 1px 5px;
  }

  .text {
    font-size: 11px;
    color: #999;
    margin-left: 13px;
    display: inline-block;
    height: 26px;
    line-height: 26px;
  }

  .textNum {
    font-size: 14px;
    color: #999;
    margin-right: 13px;
    top: 10px;
    position: absolute;
    top: 5px;
  }

}

.text-right {
  text-align: right !important;
  padding-top: 15px;

  .firstbtn {
    margin-right: 3px;
  }
}

.btn {
  display: inline-block;
  margin-bottom: 0;
  font-weight: 400;
  text-align: center;
  vertical-align: middle;
  cursor: pointer;
  background-image: none;
  border: 1px solid transparent;
  white-space: nowrap;
  padding: 6px 12px;
  font-size: 13px;
  line-height: 1.42857143;
  border-radius: 2px;
  user-select: none;
  transition: color 0.2s linear, background-color 0.2s linear, border 0.2s linear;
}

.btn-default {
  color: #333;
  background-color: #fff;
  border-color: #ccc;
}

.btn-choice {
  color: #000;

  &:hover {
    color: #fff;
    background-color: #2b80c0;
  }
}

.btn-sm {
  padding: 6px 10px 5px;
  font-size: 12px;
  line-height: 1.5;
}

.dialogtable {
  z-index: 101;

  ::v-deep {

    .el-input__inner {
      width: 266px;
      height: 32px;
    }

    .el-form-item {
      margin-bottom: 10px;
    }

    .el-form-item__label {
      margin-right: 26px;
    }

    .el-progress__text {
      display: none;
    }

    .el-progress-bar {
      padding-right: 10px;
    }
  }

  .el-progress {
    width: 197px;
    display: inline-block;

    span {
      font-size: 12px;
      color: gray;
      text-align: right;
      display: inline-block;
      width: 71px;
    }
  }
}
</style>
