<template>
    <div>
        <div>
            <!--面包屑-->
            <el-breadcrumb separator-class="el-icon-arrow-right"
                style="padding-left: 10px;padding-bottom: 10px;font-size: 12px">
                <el-breadcrumb-item :to="{ path: '/usermain' }">首页</el-breadcrumb-item>
                <el-breadcrumb-item>监控中心</el-breadcrumb-item>
                <el-breadcrumb-item>视频监控</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <video ref="videoElement" autoplay style="display: none;"></video>
        <img :src="imageSrc" class="img-styled" />
        <el-table :data="tableData" class="img-styled">
            <el-table-column prop="label" label="项目"></el-table-column>
            <el-table-column prop="value" label="数据"></el-table-column>
        </el-table>
    </div>
</template>


  
<script>
import io from 'socket.io-client';

export default {
    name: 'UserFatigueDetection',
    data() {
        return {
            videoElement: null,
            outputElement: null,
            imageSrc: '',
            fatigueCount: 0,
            status: 'Normal',
            socket: io('http://localhost:8000'), // Replace with your server URL and port
            tableData: [
                { label: '眯眼计数', value: '' },
                { label: '眼部状态', value: '' },
                { label: '打哈欠计数', value: '' },
                { label: '嘴部状态', value: '' }
            ]
        };

    },
    methods: {
        updateTableData() {
            this.tableData = [
                { label: '眯眼计数', value: this.eye_count },
                { label: '眼部状态', value: this.eye_text },
                { label: '打哈欠计数', value: this.mouth_count },
                { label: '嘴部状态', value: this.mouth_text }
            ];
        }
    },
    mounted() {
        this.videoElement = this.$refs.videoElement;
        this.outputElement = this.$refs.outputElement;

        if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
            navigator.mediaDevices.getUserMedia({ video: true }).then((stream) => {
                this.videoElement.srcObject = stream;
                this.videoElement.play();

                const canvas = document.createElement('canvas');
                const context = canvas.getContext('2d');

                canvas.width = 600; // 设置宽度为1280像素
                canvas.height = 400; // 设置高度为720像素

                setInterval(() => {
                    context.drawImage(this.videoElement, 0, 0, canvas.width, canvas.height);
                    const dataURL = canvas.toDataURL('image/jpeg');
                    this.socket.emit('frame', dataURL.split(',')[1]); // Send the frame to the server
                }, 100); // Send frames every 100ms
            });
        }

        this.socket.on('response', (data) => {
            this.imageSrc = 'data:image/jpeg;base64,' + data.data; // Update the image source
            this.eye_count = data.eye_count;
            this.eye_text = data.eye_text;
            this.mouth_count = data.mouth_count;
            this.mouth_text = data.mouth_text;
            this.updateTableData(); // 更新表格数据
        });
    },
    beforeDestroy() {
        this.socket.disconnect();
    }
};
</script>
  
<style>
.img-styled {
    border: 4px solid #ddd;
    /* 添加边框 */
    border-radius: 10px;
    /* 圆角边框 */
    box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.2);
    /* 添加阴影 */
    width: 80%;
    /* 调整图片宽度 */
    max-width: 500px;
    /* 最大宽度 */
    height: auto;
    /* 高度自适应 */
    display: block;
    /* 居中显示 */
    margin: 10px auto;
    /* 上下外边距 */
}
</style>
