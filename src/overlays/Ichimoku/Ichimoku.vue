<script>
//Ichimoku Indicator Overlay. Expected Format: [ <timestamp>, <ConversionLine>, <BaseLine>, <Lead1>, <Lead2>, <Lagging> ]
import { Overlay } from 'trading-vue-js'

export default {
    name: "Ichimoku",
    mixins: [Overlay],
    data() {
        return {
            ctxTenkan: {},
            ctxKijun: {},
            ctxSenkouSpanA: {},
            ctxSenkouSpanB: {},
            ctxChinkou: {},
            ctxFillKumo: {},
            tenkan: [],
            kijun: [],
            senkouSpanA: [],
            senkouSpanB: [],
            chinkou: [],
            tenkanLineWidth: 1,
            kijunLineWidth: 1,
            senkouSpanALineWidth: 1,
            senkouSpanBLineWidth: 1,
            chinkouLineWidth: 1,
            colorTenkan: "#52A634",
            colorKijun: "#52A59D",
            colorSenkouSpanA: "#438625",
            colorSenkouSpanB: "#bd003c",
            colorChinkou: "#BF2A64",
            colorKumoUp: "#E5F2E5",
            colorKumoDown: "#FFE5E5",
            showTenkan: true,
            showKijun: true,
            showSenkouSpanA: true,
            showSenkouSpanB: true,
            showFillKumo: true
        };
    },
    computed: {
        sett() {
            return this.$props.settings;
        },
        tenkan_color() {
            return this.sett['tenkan-color'] || this.colorTenkan;
        },
        kijun_color() {
            return this.sett['kijun-color'] || this.colorKijun;
        },
        senkou_spanA_color() {
            return this.sett['senkou_spanA_color'] || this.colorSenkouSpanA;
        },
        senkou_spanB_color() {
            return this.sett['senkou_spanB_color'] || this.colorSenkouSpanB;
        },
        chinkou_color() {
            return this.sett['chinkou_color'] || this.colorChinkou;
        },
        kumo_up_color() {
            return this.sett['kumo_up_color'] || this.colorKumoUp;
        },
        kumo_down_color() {
            return this.sett['kumo_down_color'] || this.colorKumoDown;
        },
        tenkan_line_width() {
            return this.sett['tenkan_line_width'] || this.tenkanLineWidth;
        },
        kijun_line_width() {
            return this.sett['kijun_line_width'] || this.kijunLineWidth;
        },
        senkou_spanA_line_width() {
            return this.sett['senkou_spanA_line_width'] || this.senkouSpanALineWidth;
        },
        senkou_spanB_line_width() {
            return this.sett['senkou_spanB_line_width'] || this.senkouSpanBLineWidth;
        },
        chinkou_line_width() {
            return this.sett['chinkou_line_width'] || this.chinkouLineWidth;
        },
    },
    methods: {
        calc() {
            return {  
                props: {
                    offset: { def: 26, text: 'Offset' },
                   showChinkou: { def: 0, text: 'Show Chinkou' },
                    showTenkan:  { def: 0, text: 'Show Tenkan' },
            showKijun:  { def: 0, text: 'Show Kijun' },

                },
                update: `
                    
                    let tenkan=ts((highest(high,9)[0]+lowest(low,9)[0])/2)
                    let kijun=ts((highest(high,26)[0]+lowest(low,26)[0])/2)
                    let senkouA=ts((tenkan[0]+kijun[0])/2)
                    let senkouB=ts((highest(high,52)[0]+lowest(low,52)[0])/2)
                    let final=ts([tenkan[0],kijun[0],senkouA[0], senkouB[0],high[0],senkouA[26], senkouB[26]])
                    return final
                `
            }
        },
        meta_info() {
            return {
                author: "Sudeep Batra",
                version: "1.0.0"
            };
        },
        
        draw(ctx) {
            const layout = this.$props.layout;
            const propsSub = this.$props.sub;
            console.log(this);
            console.log(ctx);
            this.ctxTenkan = ctx;
            this.ctxKijun = ctx;
            this.ctxSenkouSpanA = ctx;
            this.ctxSenkouSpanB = ctx;
            this.ctxChinkou = ctx;
            this.ctxFillKumo = ctx;
            //console.log("settings");
            //console.log(this.$props.settings);
            var subdata = this.$props.data;
            var chinkouOffset=2*this.sett.offset;
            var subdataSenkouSpan = this.$props.data //.slice(0, propsSub.length + this.sett.offset);
            var subdataChinkou = this.$props.data //.slice(0, propsSub.length - this.sett.offset);

            if (this.showFillKumo&&subdataSenkouSpan.length>0) {
                this.ctxFillKumo.beginPath();

                var ind = 0;
                var delta=0;
              
               
                var pCurr=this.map_senkou_span_values(subdataSenkouSpan,-this.sett.offset);
                var pPrev={};
                for (var i = -this.sett.offset; i < subdataSenkouSpan.length; i++) {
                        pPrev=pCurr;
                        pCurr=this.map_senkou_span_values(subdataSenkouSpan,i);
                        
                        
                        this.ctxSenkouSpanB.beginPath();
                        this.ctxSenkouSpanB.moveTo(pPrev.x, pPrev.senkouSpanA);
                        this.ctxSenkouSpanB.lineTo(pCurr.x + 0.1, pCurr.senkouSpanA);
                        this.ctxSenkouSpanB.lineTo(pCurr.x + 0.1, pCurr.senkouSpanB);
                        this.ctxSenkouSpanB.lineTo(pPrev.x, pPrev.senkouSpanB);
                        if (pPrev.senkouSpanA >= pCurr.senkouSpanB) {
                            this.ctxSenkouSpanB.fillStyle = this.kumo_down_color;
                        } else {
                            this.ctxSenkouSpanB.fillStyle = this.kumo_up_color;
                        }

                        this.ctxSenkouSpanB.fill();
                    

                    
                }
                this.ctxSenkouSpanB.stroke();
            }

            if (this.sett.showTenkan) {
                this.ctxTenkan.beginPath();
                for (var i = 0; i < subdata.length; i++) {
                    this.ctxTenkan.strokeStyle = this.tenkan_color;
                    this.ctxTenkan.lineWidth = this.tenkan_line_width;
                    this.ctxTenkan.lineTo(layout.t2screen(subdata[i][0]), layout.$2screen(subdata[i][1]));




                }

                this.ctxTenkan.stroke();
            }

            if (this.sett.showKijun) {
                this.ctxKijun.beginPath();

                for (var i = 0; i < subdata.length; i++) {
                    this.ctxKijun.strokeStyle = this.kijun_color;
                    this.ctxKijun.lineWidth = this.kijun_line_width;
                    this.ctxKijun.lineTo(layout.t2screen(subdata[i][0]), layout.$2screen(subdata[i][2]));

                }

                this.ctxKijun.stroke();
            }

            if (this.showSenkouSpanA) {
                this.ctxSenkouSpanA.beginPath();
                for (var i = -this.sett.offset; i < subdataSenkouSpan.length; i++) {

                    this.ctxSenkouSpanA.strokeStyle = this.senkou_spanA_color;
                    this.ctxSenkouSpanA.lineWidth = this.senkou_spanA_line_width;
                 
                    pCurr=this.map_senkou_span_values(subdataSenkouSpan,i);

                    this.ctxSenkouSpanA.lineTo(pCurr.x, pCurr.senkouSpanA);
                }

                this.ctxSenkouSpanA.stroke();
            }

            if (this.showSenkouSpanB) {
                this.ctxSenkouSpanB.beginPath();
                 for (var i = -this.sett.offset; i < subdataSenkouSpan.length; i++) {

                    this.ctxSenkouSpanB.strokeStyle = this.senkou_spanB_color;
                    this.ctxSenkouSpanB.lineWidth = this.senkou_spanB_line_width;
                    pCurr=this.map_senkou_span_values(subdataSenkouSpan,i);

                    this.ctxSenkouSpanB.lineTo(pCurr.x, pCurr.senkouSpanB);
                 
                }

                this.ctxSenkouSpanB.stroke();
            }

            if (this.sett.showChinkou!=0) {
                this.ctxChinkou.beginPath();
                
                for (var i = 0; i < subdataChinkou.length-chinkouOffset; i++) {
                    
                 
                       let px = layout.t2screen(subdataChinkou[i][0]);
                    
                        this.ctxChinkou.strokeStyle = this.chinkou_color;
                        this.ctxChinkou.lineWidth = this.chinkou_line_width;
                        this.ctxChinkou.lineTo(px, layout.$2screen(subdataChinkou[i+chinkouOffset][5]));
                   
                   
                }

                this.ctxChinkou.stroke();
            }
        },
        map_senkou_span_values(p,i) {
            const layout = this.$props.layout;
            let ind=i;
            let pRes=0;
            let sa=0;
            let sb=0;
            
                if( i <p.length-this.sett.offset) {
                    ind=i + this.sett.offset;

                    pRes=layout.t2screen(p[ind][0])
                }else if(i>1)
                {
                    let last=p.length;
                    pRes=layout.t2screen(p[i][0])
                    let delta=layout.t2screen(p[p.length-1][0])-layout.t2screen(p[p.length-2][0]);
                    
                    pRes=pRes+delta*this.sett.offset;

                }
            if(i<0) {
                 if (i>=p.length)
                        ind=p.length-1;
                sa=layout.$2screen(p[ind][6]),
                sb= layout.$2screen(p[ind][7])
            }else{
                sa=layout.$2screen(p[i][3]),
                sb= layout.$2screen(p[i][4])
            }
            return (
                p && {
                    x: pRes,
                    senkouSpanA:sa,
                    senkouSpanB: sb,
                }
            );
        },
        use_for() {
            return ["Ichimoku"];
        },
        data_colors() {
            let colors = [];
            colors.push(this.tenkan_color)
            colors.push(this.kijun_color)
            colors.push(this.senkou_spanA_color)
            colors.push(this.senkou_spanB_color)
            colors.push(this.chinkou_color)

            return colors;
        }
    },
};
</script>
