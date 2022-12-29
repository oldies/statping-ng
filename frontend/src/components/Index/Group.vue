<template>

    <div v-if="services.length > 0" class="col-12 full-col-12">

        <div id="accordion">
          <div class="card">
            <div class="card-header" id="headingOne">
              <h4 v-if="group.name !== 'Empty Group'" class="group_header mb-3 mt-4">{{group.name}}
                <span class="badge text-uppercase float-right" :class="{'bg-success': online, 'bg-danger': !online }">
                    {{online ? $t('online') : $t('offline')}}
                </span>
              </h4>
            </div>

            <div id="collapseOne" class="collapse show" aria-labelledby="headingOne" data-parent="#accordion">
              <div class="">


            <div v-for="(service, index) in services" v-bind:key="index" class="subitem-service">
                <router-link class="no-decoration font-3" :to="serviceLink(service)">{{service.name}}</router-link>
                <span class="badge text-uppercase float-right" :class="{'bg-success': service.online, 'bg-danger': !service.online }">
                    {{service.online ? $t('online') : $t('offline')}}
                </span>

                <GroupServiceFailures @event_child="eventChild" :service="service"/>

                <IncidentsBlock :service="service"/>

            </div>

              </div>
            </div>
          </div>
        </div>
        <br>
    </div>
</template>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.card-header h4 {
  margin-top: 0 !important;
  margin-bottom: 0 !important;
}
.subitem-service {
  padding: 0.75rem 1.25rem;
  border-bottom: 1px solid rgba(0,0,0,.125);
}
</style>

<script>
    const GroupServiceFailures = () => import(/* webpackChunkName: "index" */ './GroupServiceFailures');
    const IncidentsBlock = () => import(/* webpackChunkName: "index" */ './IncidentsBlock');

    const mergeByProperty = (target, source, prop) => {
      source.forEach(sourceElement => {
        let targetElement = target.find(targetElement => {
          return sourceElement[prop].getTime() === targetElement[prop].getTime();
        })
        if (! targetElement) {
            target.push(sourceElement);
        } else {
            targetElement['amount'] += sourceElement['amount'];
            targetElement['probes'] += sourceElement['probes'];
        }
      })
    }

export default {
  name: 'Group',
  components: {
      IncidentsBlock,
      GroupServiceFailures
  },
    data() {
        return {
            failureData: [],
          hover_text: "",
          loaded: false,
          visible: false,
          online: false,
        }
    },
  props: {
    group: {
      type: Object,
      required: true,
    }
  },
  computed: {
    services() {
      return this.$store.getters.servicesInGroup(this.group.id)
    }
  },
    methods: {
		eventChild: function(id) {
            mergeByProperty(this.failureData, id['failureData'], 'date');
            this.online = id['online'];
		}
	}
}
</script>
