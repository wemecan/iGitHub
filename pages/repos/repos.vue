<template>
	<view v-if="$_.isArray(repos)">
		<view class="order" v-for="(item, index) in repos" :key="index">
			<view class="item" @tap="clickRepoDetail(item)">
				<view class="left">
					<image v-if="item.owner" @error="imageError(item)" class="avatar" :src="item.owner.avatar_url" mode="aspectFill"></image>
				</view>
				<view class="content">
					<view v-if="item.name && item.owner" class="title u-line-2"><text selectable>{{ item.owner.login + ' / ' + item.name }}</text></view>
					<view class="description"><text selectable>{{ item.description }}</text></view>
					<u-link class="link" :font-size="30" color="#0965d2" :href="item.homepage">{{ item.homepage }}</u-link>
					<u-row class="foot">
						<u-col span="6">
							<text class="iconfont iconyuandianzhong margin-right-xs" :style="{'color':getColor(item.language)}"></text>
							<text selectable class="u-font-lg">{{item.language}}</text>
						</u-col>
						<u-col span="3">
							<text class="iconfont icongithub-star margin-right-xs text-main"></text>
							<text class="text-main u-font-lg">{{item.stargazers_count}}</text>
						</u-col>
						<u-col span="3">
							<text class="iconfont iconcode-fork margin-right-xs text-main"></text>
							<text class="text-main u-font-lg">{{item.forks_count}}</text>
						</u-col>
					</u-row>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import languageColors from '@/static/colors.json'
	export default {
		props: {
			repos: {
				type: Array,
				default: []
			}
		},
		methods: {
			imageError(item) {
				item.owner.avatar_url = '/static/img/60x60.png'
			},
			getColor(key) {
				return !this.$_.isNil(languageColors) && !this.$_.isEmpty(languageColors) && !this.$_.isNil(languageColors[key]) ?
					languageColors[key].color : ''
			},
			clickRepoDetail(item) {
				const owner = item.author ? item.author : item.owner.login
				uni.navigateTo({
					url: `/pages/repos/detail?owner=${owner}&repo=${item.name}`
				})
			}
		}
	}
</script>
