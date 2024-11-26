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
          <h4>Banco de Registros</h4>
          
          <button @click="$emit('limpiarRegistros')" type="button" class="btn btn-danger">
            Reiniciar
          </button>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R0 x</span>
            <input maxlength="4" v-model="registros[0]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R1 x</span>
            <input maxlength="4" v-model="registros[1]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R2 x</span>
            <input maxlength="4" v-model="registros[2]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R3 x</span>
            <input maxlength="4" v-model="registros[3]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R4 x</span>
            <input maxlength="4" v-model="registros[4]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R5 x</span>
            <input maxlength="4" v-model="registros[5]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R6 x</span>
            <input maxlength="4" v-model="registros[6]" type="text" class="form-control" />
          </div>
          <div class="input-group input-group-lg">
            <span class="input-group-text">R7 x</span>
            <input maxlength="4" v-model="registros[7]" type="text" class="form-control" />
          </div>
        </div>
</template>
