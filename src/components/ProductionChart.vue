<script>
    //Importing Bar class from the vue-chartjs wrapper
    import {Bar} from 'vue-chartjs';
    //Exporting this so it can be used in other components
    export default {
        extends: Bar,
        props: {
            x: {
                type: Array,
                required: false,
                default: () => Array.from({length: 20}, (_, i) => i),
            },
            y: {
                type: Array,
                required: false,
                default: () => Array.from({length: 20}, (_, i) => 100 - 5 * i),
            },
            selectXValue: {
                required: false,
                default: undefined,
            },
            minXBuffer: {
                type: Number,
                required: false,
                default: 5,
            },
            maxXBuffer: {
                type: Number,
                required: false,
                default: 10,
            },
            title: {
                type: String,
                required: false,
                default: '',
            },
            noRendering: {
                type: Boolean,
                required: false,
                default: false,
            },
        },
        computed: {
            indexOfLast100: function() {
                for (let i = 0; i < this.y.length; i++) {
                    if (this.y[i] !== 100) {
                        return i - 1;
                    }
                }
                return -1;
            },
            indexOfFirst0: function() {
                for (let i = 0; i < this.y.length; i++) {
                    if (this.y[i] === 0) {
                        return i;
                    }
                }
                return -1;
            },
            indexOfSelectedValue: function() {
                if (this.selectXValue === undefined) {
                    return -1;
                }

                return this.selectXValue;
            },
            minX: function() {
                return Math.max(this.indexOfLast100 - (this.minXBuffer - 1), 0);
            },
            maxX: function() {
                const cropPoint = Math.max(this.indexOfSelectedValue, this.indexOfFirst0);

                return Math.min(cropPoint + (this.maxXBuffer - 1), this.x.length - 1);
            },
            chartData: function() {
                return {
                    //Data to be represented on x-axis
                    labels: this.x,
                    datasets: [
                        {
                            label: 'Odds (%)',
                            backgroundColor: this.y.map((yValue, xValue) => {
                                if (xValue === this.selectXValue) {
                                    if (yValue === 100) {
                                        return 'hsla(100,50%,52%, 0.1)';
                                    }
                                    return 'hsl(100,50%,52%)';
                                }
                                return '#f87979';
                            }),
                            pointBackgroundColor: 'white',
                            borderWidth: 1,
                            pointBorderColor: '#249EBF',
                            //Data to be represented on y-axis
                            data: this.y,
                        },
                    ],
                };
            },
            options: function() {
                return {
                    title: {
                        display: !!this.title,
                        text: this.title,
                    },
                    scales: {
                        yAxes: [
                            {
                                scaleLabel: {
                                    display: true,
                                    labelString: 'Probability %',
                                },
                                ticks: {
                                    beginAtZero: false,
                                },
                                gridLines: {
                                    display: true,
                                },
                            }],
                        xAxes: [
                            {
                                scaleLabel: {
                                    display: true,
                                    labelString: 'Minimal Production Total',
                                },
                                ticks: {
                                    beginAtZero: false,
                                    min: this.minX,
                                    max: this.maxX,
                                },
                                gridLines: {
                                    display: false,
                                },
                            }],
                    },
                    legend: {
                        display: false,
                    },
                    responsive: true,
                    maintainAspectRatio: false,
                };
            },
        },
        mounted() {
            this.render();
        },
        methods: {
            render() {
                if (!this.noRendering) {
                    if (this._chart) {
                        this._chart.destroy();
                    }

                    this.renderChart(this.chartData, this.options);
                }
            },
        },
        watch: {
            x: function() {
                this.render();
            },
            y: function() {
                this.render();
            },
        },
    };
</script>
