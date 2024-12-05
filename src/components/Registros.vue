<script setup>

    const props = defineProps({
        registros:{
            type: Array,
            required: true
        },
        registrosDecConSigno:{
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

          <div class="form-floating">
            <input maxlength="4" @focus="$event.target.select()" v-model="registros[0]" type="text" class="form-control">
            <label for="floatingInput"> R0 ( {{registrosDecConSigno[0]}} )</label>
          </div>
          <div class="form-floating">
            <input maxlength="4" @focus="$event.target.select()" v-model="registros[1]" type="text" class="form-control">
            <label for="floatingInput"> R1 ( {{registrosDecConSigno[1]}} )</label>
          </div>
          <div class="form-floating">
            <input maxlength="4" @focus="$event.target.select()" v-model="registros[2]" type="text" class="form-control">
            <label for="floatingInput"> R2 ( {{registrosDecConSigno[2]}} )</label>
          </div>
          <div class="form-floating">
            <input maxlength="4" @focus="$event.target.select()" v-model="registros[3]" type="text" class="form-control">
            <label for="floatingInput"> R3 ( {{registrosDecConSigno[3]}} )</label>
          </div>
          <div class="form-floating">
            <input maxlength="4" @focus="$event.target.select()" v-model="registros[4]" type="text" class="form-control">
            <label for="floatingInput"> R4 ( {{registrosDecConSigno[4]}} )</label>
          </div>
          <div class="form-floating">
            <input maxlength="4" @focus="$event.target.select()" v-model="registros[5]" type="text" class="form-control">
            <label for="floatingInput"> R5 ({{registrosDecConSigno[5]}} )</label>
          </div>
          <div class="form-floating">
            <input maxlength="4" @focus="$event.target.select()" v-model="registros[6]" type="text" class="form-control">
            <label for="floatingInput"> R6 ( {{registrosDecConSigno[6]}} )</label>
          </div>
          <div class="form-floating">
            <input maxlength="4" @focus="$event.target.select()" v-model="registros[7]" type="text" class="form-control">
            <label for="floatingInput"> R7 ( {{registrosDecConSigno[7]}} )</label>
          </div>
       
          
        </div>
</template>
