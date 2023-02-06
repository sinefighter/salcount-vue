<template>
    <div class="wrapper">
            <div class="filter">
                <div class="date card-panel" id="filter-date">
                    <div class="row">
                        <div class="col s4">
                            от <input type="date" name="from"> 
                        </div>
                        <div class="col s4">
                            до <input type="date" name="to">
                        </div>
                        <div class="col s4">
                            <button class="waves-effect waves-light btn-small" @click="filterDate">Найти</button>
                            <a href="#" class="clear-filter" @click="this.cloneData = this.data">Очистить<i class="material-icons">clear</i></a>
                        </div>
                    </div>
                </div>
            </div>
            <div class="profit-content">
                <div class="profit-table">
                    <table>
                        <thead>
                            <tr>
                                <td class="teal lighten-4">Дата</td>
                                <td class="teal lighten-4">Проект</td>
                                <td class="teal lighten-4 sort-sum">Сумма <i class="small material-icons">import_export</i></td>
                            </tr>
                        </thead>
                        <tbody>
                            <template v-for="month in getAllMonths()" :key="month.start">
                                <tr class="row-title">
                                    <td colspan="2" class="row-month">{{ monthName(month) }}</td>
                                    <td class="row-profit">{{ monthTotal(month) }} грн</td>
                                </tr>
                                <template v-for="row in cloneData" :key="row.id">
                                    <tr v-if="isBetweenMonth(row, month)">
                                        <td><input type="date" name="date" v-model="row.date" /></td>
                                        <td><input type="text" name="project" v-model="row.project" /></td>
                                        <td><input type="number" name="profit" v-model="row.profit" /></td>
                                    </tr>
                                </template>
                            </template>
                        </tbody>
                        <tfoot>
                            <tr class="add-new">
                                <td><input type="date" name="date" v-model="newRow.date"></td>
                                <td><input type="text" name="project" v-model="newRow.project"></td>
                                <td><input type="number" name="profit" v-model="newRow.profit">
                                    <button id="add-row" class="waves-effect waves-light btn-small" @click="addRow">Добавить <i class="material-icons ">add</i></button>
                                </td>
                            </tr>
                        </tfoot>
                    </table>
                </div>
                <div class="total">
                    Всего: <b>{{ countTotal }} грн</b>
                </div>
            </div>
        </div>
    
    </template>
    
    <script>
    import moment from 'moment/moment'
    
    export default {
      name: 'App',
      components: {
      },
      data() {
        return {
            data: [],
            cloneData: [],
            newRow: {}
        }
      },
      methods: {
        addRow(e) { //добавляем новую строку
          const rowWrap = e.target.closest('.add-new')
          const inputs = rowWrap.querySelectorAll('input')
          let isComplete = true
    
          inputs.forEach((item) => { //проверка на пустоту строк
              item.classList.remove('error')
              if(item.value === '') {
                  item.classList.add('error')
                  isComplete = false
              }
          })
            // console.log(this);
    
          if(isComplete) { //если пустых строк нет - то добавляем в массив, и очищаем строку
            this.data.push(this.newRow)
            this.addToStorage()
            this.newRow = {}
          }
          console.log('addRow');
        },
    
        addToStorage() { //добавление в локалсторедж
            localStorage.setItem('data', JSON.stringify(this.data))
            console.log('addToStorage');
        },
    
        filterDate(e) { //фильтрация по дате
            const filterForm = e.target.closest('#filter-date')
            const from = filterForm.querySelector('input[name=from]').value
            const to = filterForm.querySelector('input[name=to]').value
    
            this.cloneData = this.data.filter((item) => {
                return moment(item.date).isBetween(from, to, undefined, '[]')
            })

            console.log('filterDate');
        },
    
        getAllMonths() { //получаем даты начала и конца месяцев
            moment.locale('ru');
    
            const monthArray = []
            const monthsBetween = this.cloneData.reduce((acc, item) => {
                const monthWord = moment(item.date).format('MMMM')
                if(monthArray.indexOf(monthWord) === -1) {
                    monthArray.push(monthWord)
    
                    const startOfMonth = moment(item.date).startOf('month').format('YYYY-MM-DD');
                    const endOfMonth   = moment(item.date).endOf('month').format('YYYY-MM-DD');
                    acc.push({start: startOfMonth, end: endOfMonth})
                }
                return acc
            }, [])
    
            console.log('getAllMonths');
    
            return monthsBetween
        },
    
        isBetweenMonth(row, month) { //проверяем, входит ли указанная дата в промежуток месяца
            console.log('isBetweenMonth');
            return moment(row.date).isBetween(month.start, month.end, undefined, '[]')
        },
    
        monthName(date) { //возвращает название месяца в зависимости от даты
            console.log('monthName');
            return moment(date.start).format('MMMM')
        },
    
        monthTotal(month) { //подсчет суммы заработанной за месяц
            console.log('monthTotal');
            return this.cloneData.reduce((acc, row) => {
                if(this.isBetweenMonth(row, month)) {
                    acc += row.profit
                }
                return acc
            }, 0)
        },
    
      },
      computed: {
        countTotal() { // подсчет общей суммы за все время
          return this.cloneData.reduce((acc, item) => acc + item.profit, 0)
        },
      },
      created() {
        //передаем данные в массив data
        if(localStorage.getItem('data')) {
            this.data = JSON.parse(localStorage.getItem('data'))
        }else{
          this.data = [
                {
                    date: '2023-01-01',
                    project: 'project 1',
                    profit: 10000
                },
                {
                    date: '2023-01-02',
                    project: 'project 2',
                    profit: 20000
                },
                {
                    date: '2023-01-03',
                    project: 'project 3',
                    profit: 30000
                },
                {
                    date: '2023-01-04',
                    project: 'project 4',
                    profit: 40000
                },
            ]
        }
    
        // возвращаем отсортированный массив по дате
        this.cloneData = this.data.sort(function (left, right) {
            return moment.utc(left.date).diff(moment.utc(right.date))
        });
      }
    }
    </script>
    
    <style lang="scss">
    #app {
      font-family: Avenir, Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      color: #2c3e50;
    }
    </style>
    