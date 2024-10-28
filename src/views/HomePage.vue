<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>SIMÓN</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div id="contenedor">
        <main>
          <div id="simon">
            <div class="rowSimon">
              <div :class="classRojo" class="teclasSimon do" @click="playRojoPlayer"></div>
              <div :class="classAzul" class="teclasSimon fa" @click="playAzulPlayer"></div>
            </div>
            <div class="rowSimon">
              <div :class="classVerde" class="teclasSimon re" @click="playVerdePlayer"></div>
              <div :class="classAmarillo" class="teclasSimon sol" @click="playAmarilloPlayer"></div>
            </div>
          </div>
        </main>
        <aside>
          <div id="stats">
            <ion-text color="danger">
              <h1>SIMÓN</h1>
            </ion-text>
            <ion-chip color="primary">
              <ion-label>Puntaje: {{score}}</ion-label>
            </ion-chip>
            <ion-chip color="secondary">
              <ion-label>mayor puntaje: {{highScore}}</ion-label>
            </ion-chip>
            <ion-text color="warning">
              <h3>{{score}}</h3>
            </ion-text>
            <ion-text color="danger">
              <h3 v-if="!ingame">Oprime el botón para empezar</h3>
              <h3 v-else-if="ingame && turno === 'jugador'">Repite en orden</h3>
            </ion-text>
            <ion-button v-if="!ingame" color="success" expand="block" @click="startGame">
              <ion-icon slot="start" :icon="playCircleOutline"></ion-icon>
              <ion-label>Jugar</ion-label>
            </ion-button>
            <ion-button v-else color="danger" expand="block" @click="stopGame">
              <ion-icon slot="start" :icon="stopCircleOutline"></ion-icon>
              <ion-label>Detener</ion-label>
            </ion-button>
          </div>
        </aside>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonChip, IonIcon, IonText, IonLabel } from '@ionic/vue';
import { playCircleOutline, stopCircleOutline } from 'ionicons/icons';
import { ref, watch, onMounted } from 'vue';
import doAudio from '../assets/do.mp3';
import faAudio from '../assets/fa.mp3';
import reAudio from '../assets/re.mp3';
import solAudio from '../assets/sol.mp3';
import wrongAudio from '../assets/wrong.mp3';
const classRojo = ref<string>('rojo_off');
const classAzul = ref<string>('azul_off');
const classVerde = ref<string>('verde_off');
const classAmarillo = ref<string>('amarillo_off');
let timeoutId: ReturnType<typeof setTimeout> | null = null;
const doSound:HTMLAudioElement = new Audio(doAudio);
const faSound:HTMLAudioElement= new Audio(faAudio);
const reSound:HTMLAudioElement= new Audio(reAudio);
const solSound:HTMLAudioElement= new Audio(solAudio);
const wrongSound:HTMLAudioElement= new Audio(wrongAudio);
const ingame = ref<boolean>(false);
//turnos, le toca al pc o al jugador
const turno = ref<string>('pc');
//Aquí guardamos los sonidos acumulados
const sonidos:Array<string> = [];
const opciones:Array<string> = ['do', 'fa', 're', 'sol'];
const misSonidos:Array<string> = [];
const score = ref<number>(0)
const highScore = ref<number>(0);

onMounted(() => {
  if (localStorage.getItem('highScore')) {
    highScore.value = parseInt(localStorage.getItem('highScore') as string);
  }
})

function playRojo() {
  classRojo.value = 'rojo_on';
  doSound.play();
  // Limpiar el timeout anterior si existe
  if (timeoutId) {
    clearTimeout(timeoutId);
  }
  // Cambiar el estado de clase después de un intervalo
  timeoutId = setTimeout(() => {
    classRojo.value = 'rojo_off';
    timeoutId = null; // Reiniciar el timeoutId
  }, 500); // Ajusta el tiempo según lo necesites
}

const playAzul = (): void => {
  classAzul.value = 'azul_on';
  faSound.play();
  // Limpiar el timeout anterior si existe
  if (timeoutId) {
    clearTimeout(timeoutId);
  }
  // Cambiar el estado de clase después de un intervalo
  timeoutId = setTimeout(() => {
    classAzul.value = 'azul_off';
    timeoutId = null; // Reiniciar el timeoutId
  }, 500); // Ajusta el tiempo según lo necesites

}

const playVerde = ():void => {
  classVerde.value = 'verde_on';
  reSound.play();
  // Limpiar el timeout anterior si existe
  if (timeoutId) {
    clearTimeout(timeoutId);
  }
  // Cambiar el estado de clase después de un intervalo
  timeoutId = setTimeout(() => {
    classVerde.value = 'verde_off';
    timeoutId = null; // Reiniciar el timeoutId
  }, 500); // Ajusta el tiempo según lo necesites
}

const playAmarillo = ():void =>{
  classAmarillo.value = 'amarillo_on';
  solSound.play();
  // Limpiar el timeout anterior si existe
  if (timeoutId) {
    clearTimeout(timeoutId);
  }
  // Cambiar el estado de clase después de un intervalo
  timeoutId = setTimeout(() => {
    classAmarillo.value = 'amarillo_off';
    timeoutId = null; // Reiniciar el timeoutId
  }, 500); // Ajusta el tiempo según lo necesites
}

const playRojoPc = ():void =>{
  playRojo();
}

const playAzulPc = ():void =>{
  playAzul();
}

const playVerdePc = ():void =>{
  playVerde();
}

const playAmarilloPc = ():void =>{
  playAmarillo();
}

const playRojoPlayer = ():void =>{
  playRojo();
  misSonidos.push('do')
  comparar();
}

const playAzulPlayer = ():void =>{
  playAzul();
  misSonidos.push('fa')
  comparar();
}

const playVerdePlayer = ():void =>{
  playVerde();
  misSonidos.push('re')
  comparar();
}

const playAmarilloPlayer = ():void =>{
  playAmarillo();
  misSonidos.push('sol')
  comparar();
}

const startGame = ():void => {
  ingame.value = true;
  turno.value = 'pc';
  sonidos.length = 0;
  randomSonido();
}

const stopGame = ():void => {
  ingame.value = false;
  misSonidos.length = 0;
  sonidos.length = 0;
  score.value = 0;
}


const randomSonido = (): void => {

  setTimeout(() => {
    const nuevoSonido: string = opciones[Math.floor(Math.random() * opciones.length)];
    sonidos.push(nuevoSonido);
    if (nuevoSonido === 'do') {
      playRojoPc();
    } else if (nuevoSonido === 'fa') {
      playAzulPc();
    } else if (nuevoSonido === 're') {
      playVerdePc();
    } else if (nuevoSonido === 'sol') {
      playAmarilloPc();
    }
  }, 1000);
  turno.value = 'jugador'; // Cambiar el turno al jugador
}

  watch(turno, () => {
    if (ingame.value && turno.value === 'pc') {
      randomSonido();
    }
  });

const comparar = (): void => {
  // Comprobar si los sonidos del jugador coinciden con los del PC hasta la longitud actual de misSonidos
  const sonIguales = misSonidos.every((valor, indice) => valor === sonidos[indice]);

  if (!sonIguales) {
    wrongSound.play();
    stopGame();
  } else if (misSonidos.length === sonidos.length) {
    misSonidos.length = 0; // Limpiar los sonidos del jugador para la próxima ronda
    score.value++;
    if(score.value > highScore.value){
      highScore.value = score.value;
      localStorage.setItem('highScore', highScore.value.toString());
    }
    turno.value = 'pc'; // Cambiar el turno a la computadora
  }
};

</script>

<style scoped lang="scss">
$rojo_off: #860404;
$rojo_on: #EB2B4EFF;
$azul_off: #070791;
$azul_on: #5555e7;
$verde_off: #025302;
$verde_on: #00FF00;
$amarillo_off: #a46d06;
$amarillo_on: #f9c873;

#contenedor {
  display: flex;
  align-items: center;
  flex-direction: row;
  margin: 0 auto;
  max-width: 70rem;
}

main {
  width: 60%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  padding: 5rem;
}

aside {
  width: 40%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  text-align: center;
  & h1 {
    font-size: 5rem;
    margin-bottom: 1rem;
  }
  & h2 {
    font-size: 3.5rem;
    margin-bottom: 0.5rem;
  }
  & h3 {
    font-size: 2rem;
    margin-bottom: 0.5rem;
  }
}

#simon {
  padding: 2rem;
}

.rowSimon {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-direction: row;
  width: 100%;
  height: 100%;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
}

.teclasSimon {
  width: 300px;
  height: 300px;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.7);
  transition: all 0.2s ease; /* Añadir transición suave */
}

.do {
  border-top-left-radius: 100%;
}

.fa {
  border-top-right-radius: 100%;
}

.re {
  border-bottom-left-radius: 100%;
}

.sol {
  border-bottom-right-radius: 100%;
}

.rojo_off {
  background-color: $rojo_off;
}

.rojo_on {
  background-color: $rojo_on;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.9); /* Sombra más pequeña */
  transform: translateY(4px); /* Mover hacia abajo */
}
.azul_off {
  background-color: $azul_off;
}

.azul_on {
  background-color: $azul_on;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.9); /* Sombra más pequeña */
  transform: translateY(4px); /* Mover hacia abajo */
}

.verde_off {
  background-color: $verde_off;
}

.verde_on {
  background-color: $verde_on;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.9); /* Sombra más pequeña */
  transform: translateY(-4px); /* Mover hacia abajo */
}

.amarillo_off {
  background-color: $amarillo_off;
}

.amarillo_on {
  background-color: $amarillo_on;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.9); /* Sombra más pequeña */
  transform: translateY(-4px); /* Mover hacia abajo */
}

ion-button {
  margin-top: 4rem;
}

@media (max-width: 768px) {

  #contenedor {
    flex-direction: column;
    margin: 0 auto;
  }
  main {
    width: 100%;
    margin-top: 30px;
    padding: 0;
  }
  aside {
    width: 100%;
    height: 100%;
    & h1 {
      font-size: 2rem;
      display: none;
    }
    & h2 {
      font-size: 1.5rem;
      margin-bottom: 0.5rem;
    }
    & h3 {
      font-size: 1rem;
      margin-bottom: 0.5rem;
      display: none;
    }
  }
  .rowSimon {
    max-width: 100%;
    justify-content: center;
    gap: 1rem;
  }

  .teclasSimon {
    width: 130px;
    height: 130px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  ion-button {
    margin-top: 1rem;
  }
  .do {
    border-top-left-radius: 100%;
  }

  .fa {
    border-top-right-radius: 100%;
  }

  .re {
    border-bottom-left-radius: 100%;
  }

  .sol {
    border-bottom-right-radius: 100%;
  }
}

</style>
