<script setup>

    const props = defineProps({
        registros:{
            type: Array,
            required: true
        }
    })

    defineEmits(['limpiarRegistros'])

</script>

<template>
      <div class="d-grid gap-2">
          <h4>Banco de Registros en Hexadecimal</h4>
          
          <button @click="$emit('limpiarRegistros')" type="button" class="btn btn-danger">
            Reiniciar registros
          </button>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R0</span>
            <input v-model="registros[0]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R1</span>
            <input v-model="registros[1]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R2</span>
            <input v-model="registros[2]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R3</span>
            <input v-model="registros[3]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R4</span>
            <input v-model="registros[4]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R5</span>
            <input v-model="registros[5]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R6</span>
            <input v-model="registros[6]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R7</span>
            <input v-model="registros[7]" type="text" class="form-control" />
          </div>
        </div>
</template>
