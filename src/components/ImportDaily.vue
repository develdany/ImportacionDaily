<template>
    <section>
        <div class="content">
                <b-card 
                    header-bg-variant="dark" 
                    header-text-variant="white"
                    header="Modulo de Importacion Daily Report - SIREP" 
                    class="text-center m-4"
                >
                    <input type="file" class="mb-3" id="archivoExcel" @change="subirExcel()" />

                    <b-alert :show=show variant="success">Archivo cargado con éxito. <b>Ingresar fechas para filtrado:</b></b-alert>

                    <b-row v-if="mostrar">
                        <b-col offset="3" cols="3">
                           <label for="example-datepicker">Fecha desde:</label>
                           <b-form-datepicker id="fechaini" v-model="fini" class="mb-2"></b-form-datepicker>
                        </b-col>
                        <b-col cols="3">
                           <label for="example-datepicker">Fecha hasta:</label>
                           <b-form-datepicker id="fechafin" v-model="ffin" class="mb-2"></b-form-datepicker>
                        </b-col>
                        <b-row class="mt-3">
                            <b-col class="text-center"> 
                                <b-button 
                                    variant="primary"
                                    @click="generar"
                                    class="mr-3"
                                >
                                    Generar Datos
                                </b-button>
                                <export-excel
                                    class   = "btn btn-success"
                                    :fields = "encExport"
                                    :data   = "datos"
                                    name    = "conciliado.xls"
                                    v-if="mostrarExcel"
                                >
                                    Download Excel
                                </export-excel>
                            </b-col>
                        </b-row>
                    </b-row>

                    <b-row v-if="mostrarTabla" class="mt-3">
                        <b-table  
                            :items="datos" 
                            :fields="encabezado"
                            sticky-header
                            striped
                            small
                            fixed
                            no-border-collapse
                        >
                        </b-table>
                    </b-row>
                </b-card>
          </div>
    </section>
</template>
<script>
// import XlsxRead from "vue-xlsx/dist/components/XlsxRead";
// import XlsxJson from "vue-xlsx/dist/components/XlsxJson";
import readXlsFile from "read-excel-file";
import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-vue/dist/bootstrap-vue.css'
import moment from 'moment'

export default {
  components: {

    // XlsxRead,
    // XlsxJson,
  },
  data() {
    return {
      mostrarExcel:false,
      datos:[],
      mostrarTabla:false,
      fini:Date(),
      ffin:Date(),
      show:false,
      mostrar: false,
      stickyHeader:true,
      columnas:[],
      items: [],
      encabezado:[
          {
            key: 'id',
            name:'ID-VARIABLE',
          },
          {
            key:'fecha',
            name:'FECHA',
          },
          {
            key:'valor',
            name:'VALOR',
            formatter: (value) => {
              return parseFloat(value).toFixed(2);
            }
          },
          {
            key:'estado',
            name:'ESTADO'
          }
      ],
      encExport:{
        'ID':'id',
        'VALOR':{
                  field: 'valor',
                  callback: (value) => {
                      return parseFloat(value).toFixed(2);
                  }
        },
        'FECHA':'fecha',
        'ESTADO':'estado'
      }
    };
  },

  methods: {
    subirExcel(){
        const input = document.getElementById('archivoExcel');
        const schema = {
          //Nombre en el archivo excel : Nombre que sale en el Objeto
          'Fecha':{
            prop:'Fecha',
            type:Date,
          },
          'TP-MineralTriturado':{
            prop:'TP-MineralTriturado',
            type: String
          },
          'TP-LeyAu':{
            prop:'TP-LeyAu',
            type: String
          },
          'TP-AuTriturado':{
            prop:'TP-AuTriturado',
            type: String
          },
          'TP-Productividad':{
            prop:'TP-Productividad',
            type: String
          },
          'TP-Disponibilidad':{
            prop:'TP-Disponibilidad',
            type: String
          },
          'TP-Utilizacion':{
            prop:'TP-Utilizacion',
            type: String
          },
          'TP-HorasOperativas':{
            prop:'TP-HorasOperativas',
            type: String
          },
          'HPGR-MineralTriturado':{
            prop:'HPGR-MineralTriturado',
            type: String
          },
          'HPGR-LeyAu':{
            prop:'HPGR-LeyAu',
            type: String
          },
          'HPGR-AuTriturado':{
            prop:'HPGR-AuTriturado',
            type: String
          },
          'HPGR-Productividad':{
            prop:'HPGR-Productividad',
            type: String
          },
          'HPGR-Disponibilidad':{
            prop:'HPGR-Disponibilidad',
            type: String
          },
          'HPGR-Utilizacion':{
            prop:'HPGR-Utilizacion',
            type: String
          },
          'HPGR-P80':{
            prop:'HPGR-P80',
            type: String
          },
          'HPGR-HorasOperativas':{
            prop:'HPGR-HorasOperativas',
            type: String
          },
          'AGLOM-MineralAglomerado':{
            prop:'AGLOM-MineralAglomerado',
            type: String
          },
          'AGLOM-Productividad':{
            prop:'AGLOM-Productividad',
            type: String
          },
          'AGLOM-Utilizacion':{
            prop:'AGLOM-Utilizacion',
            type: String
          },
          'AGLOM-Disponibilidad':{
            prop:'AGLOM-Disponibilidad',
            type: String
          },
          'AGLOM-AdicionCemento':{
            prop:'AGLOM-AdicionCemento',
            type: String
          },
          'AGLOM-AdicionCN':{
            prop:'AGLOM-AdicionCN',
            type: String
          },
          'AGLOM-Humedad':{
            prop:'AGLOM-Humedad',
            type: String
          },
          'AGLOM-HorasOperativas':{
            prop:'AGLOM-HorasOperativas',
            type: String
          },
          'AGLOM-Cemento':{
            prop:'AGLOM-Cemento',
            type: String
          },
          'STACKER-MineralStacker':{
            prop:'STACKER-MineralStacker',
            type: String
          },
          'STACKER-LeyAu':{
            prop:'STACKER-LeyAu',
            type: String
          },
          'STACKER-AuApiladoStacker':{
            prop:'STACKER-AuApiladoStacker',
            type: String
          },
          'STACKER-Recuperacion':{
            prop:'STACKER-Recuperacion',
            type: String
          },
          'STACKER-AuExtraibleApilado':{
            prop:'STACKER-AuExtraibleApilado',
            type: String
          },
          'STACKER-Productividad':{
            prop:'STACKER-Productividad',
            type: String
          },
          'STACKER-Disponibilidad':{
            prop:'STACKER-Disponibilidad',
            type: String
          },
          'STACKER-Utilizacion':{
            prop:'STACKER-Utilizacion',
            type: String
          },
          'STACKER-HorasOperativas':{
            prop:'STACKER-HorasOperativas',
            type: String
          },
          'AP-MineralApilado':{
            prop:'AP-MineralApilado',
            type: String
          },
          'AP-LeyAu':{
            prop:'AP-LeyAu',
            type: String
          },
          'AP-TotalAuApilado':{
            prop:'AP-TotalAuApilado',
            type: String
          },
          'AP-Recuperacion':{
            prop:'AP-Recuperacion',
            type: String
          },
          'AP-TotalAuExApilado':{
            prop:'AP-TotalAuExApilado',
            type: String
          },
          'LIXI-SolucionBarren':{
            prop:'LIXI-SolucionBarren',
            type: String
          },
          'LIXI-CNSolucionBarren':{
            prop:'LIXI-CNSolucionBarren',
            type: String
          },
          'LIXI-SolucionILS':{
            prop:'LIXI-SolucionILS',
            type: String
          },
          'LIXI-CNSolucionILS':{
            prop:'LIXI-CNSolucionILS',
            type: String
          },
          'LIXI-SolucionPLS':{
            prop:'LIXI-SolucionPLS',
            type: String
          },
          'LIXI-LeyAuSolucionPLS':{
            prop:'LIXI-LeyAuSolucionPLS',
            type: String
          },
          'LIXI-AuLixiviado':{
            prop:'LIXI-AuLixiviado',
            type: String
          },
          'LIXI-CNSolucionPLS':{
            prop:'LIXI-CNSolucionPLS',
            type: String
          },
          'LIXI-pHSolucionPLS':{
            prop:'LIXI-pHSolucionPLS',
            type: String
          },
          'SART-PLSaSART':{
            prop:'SART-PLSaSART',
            type: String
          },
          'SART-LeyCuAlimentada':{
            prop:'SART-LeyCuAlimentada',
            type: String
          },
          'SART-LeyCuSalida':{
            prop:'SART-LeyCuSalida',
            type: String
          },
          'SART-LeyAuAlimentada':{
            prop:'SART-LeyAuAlimentada',
            type: String
          },
          'SART-LeyAuSalida':{
            prop:'SART-LeyAuSalida',
            type: String
          },
          'SART-Eficiencia':{
            prop:'SART-Eficiencia',
            type: String
          },
          'ADR-PLSaCarbones':{
            prop:'ADR-PLSaCarbones',
            type: String
          },
          'ADR-LeyAuPLS+ILS':{
            prop:'ADR-LeyAuPLS+ILS',
            type: String
          },
          'ADR-LeyAuBLS':{
            prop:'ADR-LeyAuBLS',
            type: String
          },
          'ADR-Eficiencia':{
            prop:'ADR-Eficiencia',
            type: String
          },
          'ADR-AuAdsorbido':{
            prop:'ADR-AuAdsorbido',
            type: String
          },
          'ADR-AuDesorbido':{
            prop:'ADR-AuDesorbido',
            type: String
          },
          'ADR-AuDore':{
            prop:'ADR-AuDore',
            type: String
          },
        }
        readXlsFile(input.files[0] , { schema }).then(({ rows,column }) => {
          this.items = rows;
          this.columnas = column;
          console.log('rooooow',rows);
          this.show=true;
          this.mostrar=true;
        })
    },
    generar(){
        let count=0;
        let fi = moment(this.fini).format('YYYY-MM-DD');
        let ff = moment(this.ffin).format('YYYY-MM-DD');
        //console.log('fecha ini',fi);
        //console.log('fecha fin',ff);
        this.items.forEach( fila => {
          let fec = moment(fila.Fecha).add(1,'days').format('YYYY-MM-DD');
          //console.log('fecha arreglo',fec);
          
          if ( fec>=fi && fec<=ff) {
              count++;
              let obj = {};
              for (const prop in fila) {
                  //console.log(prop,fila[prop]);
                  
                  switch (prop) {
                    case 'TP-MineralTriturado':
                          obj.id=10005;
                          obj.valor = fila['TP-MineralTriturado'];
                      break;
                    case 'TP-LeyAu':
                          obj.id=10004;
                          obj.valor = fila['TP-LeyAu'];
                      break;
                    case 'TP-AuTriturado':
                          obj.id=10002;
                          obj.valor = fila['TP-AuTriturado'];
                      break;
                    case 'TP-Productividad':
                          obj.id=10006;
                          obj.valor = fila['TP-Productividad'];
                      break;
                    case 'TP-Disponibilidad':
                          obj.id=10003;
                          obj.valor = fila['TP-Disponibilidad'];
                      break;
                    case 'TP-Utilizacion':
                          obj.id=10007;
                          obj.valor = fila['TP-Utilizacion'];
                      break;
                    case 'TP-HorasOperativas':
                          obj.id=10062;
                          obj.valor = fila['TP-HorasOperativas'];
                      break;
                    case 'HPGR-MineralTriturado':
                          obj.id=10011;
                          obj.valor = fila['HPGR-MineralTriturado'];
                      break;
                    case 'HPGR-LeyAu':
                          obj.id=10010;
                          obj.valor = fila['HPGR-LeyAu'];
                      break;
                    case 'HPGR-AuTriturado':
                          obj.id=10008;
                          obj.valor = fila['HPGR-AuTriturado'];
                      break;
                    case 'HPGR-Productividad':
                          obj.id=10013;
                          obj.valor = fila['HPGR-Productividad'];
                      break;
                    case 'HPGR-Disponibilidad':
                          obj.id=10009;
                          obj.valor = fila['HPGR-Disponibilidad'];
                      break;
                    case 'HPGR-Utilizacion':
                          obj.id=10014;
                          obj.valor = fila['HPGR-Utilizacion'];
                      break;
                    case 'HPGR-P80':
                          obj.id=10012;
                          obj.valor = fila['HPGR-P80'];
                      break;
                    case 'HPGR-HorasOperativas':
                          obj.id=10063;
                          obj.valor = fila['HPGR-HorasOperativas'];
                      break;
                    case 'AGLOM-MineralAglomerado':
                          obj.id=10019;
                          obj.valor = fila['AGLOM-MineralAglomerado'];
                      break;
                    case 'AGLOM-Productividad':
                          obj.id=10020;
                          obj.valor = fila['AGLOM-Productividad'];
                      break;
                    case 'AGLOM-Utilizacion':
                          obj.id=10021;
                          obj.valor = fila['AGLOM-Utilizacion'];
                      break;
                    case 'AGLOM-Disponibilidad':
                          obj.id=10017;
                          obj.valor = fila['AGLOM-Disponibilidad'];
                      break;
                    case 'AGLOM-AdicionCemento':
                          obj.id=10015;
                          obj.valor = fila['AGLOM-AdicionCemento'];
                      break;
                    case 'AGLOM-AdicionCN':
                          obj.id=10016;
                          obj.valor = fila['AGLOM-AdicionCN'];
                      break;
                    case 'AGLOM-Humedad':
                          obj.id=10018;
                          obj.valor = fila['AGLOM-Humedad'];
                      break;
                    case 'AGLOM-Cemento':
                          obj.id=10067;
                          obj.valor = fila['AGLOM-Cemento'];
                      break;
                    case 'AGLOM-HorasOperativas':
                          obj.id=10064;
                          obj.valor = fila['AGLOM-HorasOperativas'];
                      break;
                    case 'STACKER-MineralStacker':
                          obj.id=10031;
                          obj.valor = fila['STACKER-MineralStacker'];
                      break;
                    case 'STACKER-LeyAu':
                          obj.id=10030;
                          obj.valor = fila['STACKER-LeyAu'];
                      break;
                    case 'STACKER-AuApiladoStacker':
                          obj.id=10027;
                          obj.valor = fila['STACKER-AuApiladoStacker'];
                      break;
                    case 'STACKER-Recuperacion':
                          obj.id=10033;
                          obj.valor = fila['STACKER-Recuperacion'];
                      break;
                    case 'STACKER-AuExtraibleApilado':
                          obj.id=10028;
                          obj.valor = fila['STACKER-AuExtraibleApilado'];
                      break;
                    case 'STACKER-Productividad':
                          obj.id=10032;
                          obj.valor = fila['STACKER-Productividad'];
                      break;
                    case 'STACKER-Disponibilidad':
                          obj.id=10029;
                          obj.valor = fila['STACKER-Disponibilidad'];
                      break;
                    case 'STACKER-Utilizacion':
                          obj.id=10034;
                          obj.valor = fila['STACKER-Utilizacion'];
                      break;
                    case 'STACKER-HorasOperativas':
                          obj.id=10065;
                          obj.valor = fila['STACKER-HorasOperativas'];
                      break;
                    case 'AP-MineralApilado':
                          obj.id=10039;
                          obj.valor = fila['AP-MineralApilado'];
                      break;
                    case 'AP-LeyAu':
                          obj.id=10035;
                          obj.valor = fila['AP-LeyAu'];
                      break;
                    case 'AP-TotalAuApilado':
                          obj.id=10037;
                          obj.valor = fila['AP-TotalAuApilado'];
                      break;
                    case 'AP-Recuperacion':
                          obj.id=10036;
                          obj.valor = fila['AP-Recuperacion'];
                      break;
                    case 'AP-TotalAuExApilado':
                          obj.id=10038;
                          obj.valor = fila['AP-TotalAuExApilado'];
                      break;
                    case 'LIXI-SolucionBarren':
                          obj.id=10059;
                          obj.valor = fila['LIXI-SolucionBarren'];
                      break;
                    case 'LIXI-CNSolucionBarren':
                          obj.id=10055;
                          obj.valor = fila['LIXI-CNSolucionBarren'];
                      break;
                    case 'LIXI-SolucionILS':
                          obj.id=10060;
                          obj.valor = fila['LIXI-SolucionILS'];
                      break;
                    case 'LIXI-CNSolucionILS':
                          obj.id=10056;
                          obj.valor = fila['LIXI-CNSolucionILS'];
                      break;
                    case 'LIXI-SolucionPLS':
                          obj.id=10061;
                          obj.valor = fila['LIXI-SolucionPLS'];
                      break;
                    case 'LIXI-LeyAuSolucionPLS':
                          obj.id=10057;
                          obj.valor = fila['LIXI-LeyAuSolucionPLS'];
                      break;
                    case 'LIXI-AuLixiviado':
                          obj.id=10053;
                          obj.valor = fila['LIXI-AuLixiviado'];
                      break;
                    case 'LIXI-CNSolucionPLS':
                          obj.id=10054;
                          obj.valor = fila['LIXI-CNSolucionPLS'];
                      break;
                    case 'LIXI-pHSolucionPLS':
                          obj.id=10058;
                          obj.valor = fila['LIXI-pHSolucionPLS'];
                      break;
                    case 'SART-PLSaSART':
                          obj.id=10045;
                          obj.valor = fila['SART-PLSaSART'];
                      break;
                    case 'SART-LeyCuAlimentada':
                          obj.id=10043;
                          obj.valor = fila['SART-LeyCuAlimentada'];
                      break;
                    case 'SART-LeyCuSalida':
                          obj.id=10044;
                          obj.valor = fila['SART-LeyCuSalida'];
                      break;
                    case 'SART-LeyAuAlimentada':
                          obj.id=10041;
                          obj.valor = fila['SART-LeyAuAlimentada'];
                      break;
                    case 'SART-LeyAuSalida':
                          obj.id=10042;
                          obj.valor = fila['SART-LeyAuSalida'];
                      break;
                    case 'SART-Eficiencia':
                          obj.id=10040;
                          obj.valor = fila['SART-Eficiencia'];
                      break;
                    case 'ADR-PLSaCarbones':
                          obj.id=10052;
                          obj.valor = fila['ADR-PLSaCarbones'];
                      break;
                    case 'ADR-LeyAuPLS+ILS':
                          obj.id=10051;
                          obj.valor = fila['ADR-LeyAuPLS+ILS'];
                      break;
                    case 'ADR-LeyAuBLS':
                          obj.id=10050;
                          obj.valor = fila['ADR-LeyAuBLS'];
                      break;
                    case 'ADR-Eficiencia':
                          obj.id=10049;
                          obj.valor = fila['ADR-Eficiencia'];
                      break;
                    case 'ADR-AuAdsorbido':
                          obj.id=10046;
                          obj.valor = fila['ADR-AuAdsorbido'];
                      break;
                    case 'ADR-AuDesorbido':
                          obj.id=10047;
                          obj.valor = fila['ADR-AuDesorbido'];
                      break;
                    case 'ADR-AuDore':
                          obj.id=10048;
                          obj.valor = fila['ADR-AuDore'];
                      break;
                    default:
                      break;
                  }
                  if (prop !== 'Fecha') {
                    obj.fecha=fec;
                    obj.estado=1;
                    this.datos.push(obj);
                    obj = {};
                  }                    
              }
          }
        });
        console.log('Contador',count);
        console.log('DAtosFinales',this.datos);
        this.mostrarTabla = true;
        this.mostrarExcel = true;
    }
  },
}
</script>



