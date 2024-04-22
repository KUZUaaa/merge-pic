
<script>
import DDR from 'yoyoo-ddr-vue2'
import 'yoyoo-ddr-vue2/dist/yoyoo-ddr.css'

export default {
  props:{
    url:{
      type:String,
      default:''
    },
    obj:{
      type:Object,
      default:()=>{
        return {
          transform: { x: 463, y: 376, width: 304, height: 304, rotation: 0 },
          active: true,
          rotatable: true,
          draggalbe: true,
          resizable: true,
          parent: true,
          resizeHandler: ['tl', 'tr', 'br', 'bl'],
          minWidth: 10,
          minHeight: 10,
          acceptRatio: true,
        }
    }
  }
  },
  data() {
    return {
    };
  },
  methods: {
    change(event,transform){
      this.obj.transform.x = transform.x;
      this.obj.transform.y = transform.y;
      this.obj.transform.width = transform.width;
      this.obj.transform.height = transform.height;
      this.obj.transform.rotation = transform.rotation;
    },
    onDrag(event, transform) {
      this.change(event, transform)
    },
    onDragStart(event, transform) {
      this.change(event, transform)
    },
    onDragEnd(event, transform) {
      this.change(event, transform)
    },

    onResize(event, transform) {this.change(event, transform)},
    onResizeStart(event, transform) {this.change(event, transform)},
    onResizeEnd(event, transform) {this.change(event, transform)},

    onRotate(event, transform) {this.change(event, transform)},
    onRotateStart(event, transform) {this.change(event, transform)},
    onRotateEnd(event, transform) {this.change(event, transform)},

    beforeActive(id) {
      // cell-id
      return true
    },

    renderContent(cell) {
      // cell instance of DDR
      return <img src={this.url} style="width:100%;height:100%;" />
    },
  },
  render() {
    return (
      <DDR
        active={this.obj.active}
        draggable={this.obj.draggable}
        resizable={this.obj.resizable}
        rotatable={this.obj.rotatable}
        parent={this.obj.parent}
        minWidth={this.obj.minWidth}
        minHeight={this.obj.minHeight}
        acceptRatio={this.obj.acceptRatio}
        onDrag={this.onDrag}
        onDragstart={this.onDragStart}
        onDragend={this.onDragEnd}
        onResize={this.onResize}
        onResizestart={this.onResizeStart}
        onResizeEnd={this.onResizeEnd}
        onRotate={this.onRotate}
        onRotatestart={this.onRotateStart}
        onRotatend={this.onRotateEnd}
        value={this.obj.transform}
        id={'cell-id'}
        beforeActive={this.beforeActive}
        // renderContent={this.renderContent}
      >
      <img src={this.url} style="width:100%;height:100%;object-fit:contain;" />
      </DDR>
    )
  }
};
</script>

<style>

</style>