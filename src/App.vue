<template>
  <div id="app">
    <div
        class="box"
        @mousedown="dragStart"
        :style="boxStyle"
    />
  </div>
</template>

<script>
  import { fromEvent, merge, of } from 'rxjs'
  import { switchMap, takeUntil } from 'rxjs/operators'

  export default {
    name: 'app',
    observableMethods: [
      'dragStart'
    ],
    subscriptions () {
      return {
        position: this.dragStart$.pipe(
          switchMap((e) => {
            const { left, top } = e.target.getBoundingClientRect()
            return of([left, top, e.clientX, e.clientY])
          }),
          switchMap(([left, top, x, y]) => {
            return merge(
              this.$fromDOMEvent('.box', 'mousemove'),
              fromEvent(document, 'mousemove')
            ).pipe(
              switchMap((e) => of([left, top, e.clientX - x, e.clientY - y])),
              takeUntil(merge(
                this.$fromDOMEvent('.box', 'mouseup'),
                fromEvent(document, 'mouseup')
              ))
            )
          })
        )
      }
    },
    computed: {
      boxStyle () {
        const [left, top, dx, dy] = this.position || [0, 0, 0, 0]
        return {
          top: `${top + dy}px`,
          left: `${left + dx}px`
        }
      }
    }
  }
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

  .box {
    position: absolute;
    top: 0;
    left: 0;
    width: 100px;
    height: 100px;
    background-color: cornflowerblue;
  }
</style>
