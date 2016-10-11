<template>
    <div class="ui container">
        <div class="ui segments">
            <div class="ui segment">
                <p>Overview</p>
            </div>
            <div class="ui secondary segment">
                <!--<div class="ui active inverted dimmer" v-if="!finished">
                    <div class="ui active inverted loader"></div>
                </div>-->
                <div class="ui two center aligned statistics">
                    <div class="statistic">
                        <div class="value" v-text="guild.name"></div>
                        <div class="label" v-text="guild.realm"></div>
                    </div>
                    <div class="statistic">
                        <div class="value" v-text="averageItemLevel"></div>
                        <div class="label">Average Item Level</div>
                    </div>
                </div>

                <!--<div class="ui bottom attached orange progress" v-if="!finished">
                    <div class="bar" :style="{ width: progress + '%' }"></div>
                </div>-->
            </div>
        </div>
        <div class="ui mobile reversed  stackable two column centered grid">
            <div class="twelve wide column">
                <div class="ui segment">
                    <div class="ui large middle aligned divided list">
                        <guild-member v-for="member in members" :name="member.name" :realm="member.realm">
                        </guild-member>
                    </div>
                </div>
            </div>
            <div class="four wide column">
                <h4 class="ui horizontal divider header"><i class="sort alphabet ascending icon"></i>Order By</h4>
                <div class="ui fluid vertical menu">
                    <a class="item" :class="{ active: sortBy == 'rank' }" v-on:click="sort('rank', 'name', $event)">
                        Guild Rank
                        <i class="dropdown icon" :class="{ 'vertically flipped': !sortReverse }" v-if="sortBy == 'rank'"></i>
                    </a>
                    <a class="item" :class="{ active: sortBy == 'name' }" v-on:click="sort('name', 'rank', $event)">
                        Name
                        <i class="dropdown icon" :class="{ 'vertically flipped': !sortReverse }" v-if="sortBy == 'name'"></i>
                    </a>
                    <a class="item" :class="{ active: sortBy == 'averageItemLevelEquipped' }" v-on:click="sort('averageItemLevelEquipped', 'name', $event)">
                        Item Level
                        <i class="dropdown icon" :class="{ 'vertically flipped': !sortReverse }" v-if="sortBy == 'averageItemLevelEquipped'"></i>
                    </a>
                </div>

                <h4 class="ui horizontal divider header"><i class="filter icon"></i>Filter</h4>
                <div class="ui segment">
                    <form class="ui form">
                        <div class="field">
                            <label>Name</label>
                            <input type="text" placeholder="Aeshwyn" v-model="filterName">
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import GuildMember from '../components/GuildMember';

function sortByProperty(members, prop, secondprop, reverse) {
    var sortedMembers = members.sort(function(a, b) { 
        if (a[prop] < b[prop]) {
            return -1
        } else if (a[prop] > b[prop]) {
            return 1
        } else {
            //Equal - use second prop
            return (a[secondprop] < b[secondprop] ? -1 : 1);
        }
    });
    return (reverse) ? sortedMembers.reverse() : sortedMembers;
}

var data = {
    sortBy: 'rank',
    sortBySecond: 'name',
    sortReverse: false,
    filterName: ''
}

export default {

    data: function () {
        return data;
    },

    computed: {
        guild: function() {
            return this.$store.state.guild;
        },
        
        averageItemLevel: function() {
            var avg = 0;
            this.$store.state.guild.members.forEach(function(element) {
                avg += element.averageItemLevelEquipped;
            });
            avg = Math.round(avg / this.$store.state.guild.members.length);
            return (typeof avg === 'number' && !isNaN(avg)) ? avg : 0;
        },

        members: function() {
            var guildMembers = this.guild.members;
            return sortByProperty(guildMembers, this.sortBy, this.sortBySecond, this.sortReverse)
                .filter(function(x) {
                    return (x.name.indexOf(this.filterName) !== -1)
                }, this);
        }
    },

    mounted: function() {
        this.$store.dispatch('updateGuild', { realm: this.$route.params.realm, name: this.$route.params.guild});
    },

    methods: {

        addItemLevel: function(itemLevel) {
            this.itemLevels.push(itemLevel);
        },

        sort: function(prop, secondprop) {
            if (this.sortBy === prop && this.sortBySecond === secondprop) {
                this.sortReverse = !this.sortReverse;
            } else {
                this.sortBy = prop;
                this.sortBySecond = secondprop;
                this.sortReverse = false;
            }
        }
    },

    components: {
        'guild-member': GuildMember
    }
};
</script>