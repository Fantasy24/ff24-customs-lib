<template>
  <div>
    <el-row>
      <el-tag effect="plain" type="danger">CHÍNH SÁCH BẢO MẬT!</el-tag>
    </el-row>
    <el-row style="color: #1890ff; margin-left: 3%;">
      <p>Một mật khẩu (password) mạnh phải thỏa mãn :</p>
      <p>- Dài từ 8 ký tự trở lên.</p>
      <p>- Thỏa mãn các điều kiện dưới:</p>
      <p class="indent-text">- Chứa từ 3 trong 4 loại ký tự sau: chữ hoa (A, B, C…), chữ thường (a, b, c), chữ số (0, 1, 2...), ký
        tự đặc biệt (@#...)</p>
      <p class="indent-text">- Mật khẩu không được có phần trùng với account</p>
      <p class="indent-text">- Mật khẩu mới không được trùng với 2 lần mật khẩu gần nhất</p>
      <p>- Ví dụ: Mật khẩu mạnh : Hq123456, etc2020@...</p>
    </el-row>
  </div>
</template>

<script>
export default {
  name: 'RuleChangePass'
}
</script>

<style scoped>
.indent-text {
  text-indent: 30px;
}
</style>
