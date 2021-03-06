<!-- repo detail -->
<template>
	<view :class="darkMode?'custom-dark':'custom-light'">
		<scroll-view scroll-y class="u-skeleton">
			<view class="order u-skeleton-rect">
				<view class="item">
					<view class="left">
						<image @error="imageError" class="avatar" :src="repo.owner.avatar_url" mode="aspectFill"></image>
					</view>
					<view class="content">
						<view style="flex-flow: row wrap; justify-content: space-between; align-items: center; display: flex;">
							<view class="title u-line-2"><text selectable>{{ repo.full_name }}</text></view>
						</view>
						<view class="description"><text selectable>{{ repo.description }}</text></view>
						<u-link v-if="repo.homepage" class="link" :font-size="30" color="#0965d2" :href="repo.homepage">{{ repo.homepage }}</u-link>
						<view v-if="repo.updated_at" class="remark">{{ $t('Updated') + ' ' + $u.timeFrom(new Date(repo.updated_at).getTime(), 'yyyy-mm-dd hh:MM') }}</view>
					</view>
				</view>
			</view>
			<view class="cu-list grid col-3 text-center no-border u-skeleton-rect">
				<view class="cu-item" v-for="(value, key) in numInfo" :key="key">
					<view class="badge text-xxl grid-text">
						<block>{{value}}</block>
					</view>
					<text class="text-xl">{{key}}</text>
				</view>
			</view>
			<u-gap />
			<view class="cu-progress radius u-skeleton-rect" @tap="clickLangPercent">
				<view v-for="item in reposLanguages" :key="item.lang" :style="[{ width: item.percent, 'background-color': item.color}]"></view>
			</view>

			<view class="cu-list menu u-skeleton-rect">
				<!-- code -->
				<view class="cu-item arrow" @tap="tapCode">
					<view class="content">
						<text class="list-left-icon cu-avatar round iconfont iconcode" :style="{'background-color':languageColor}" />
						<text class="padding-left-sm">{{repo.language}}</text>
					</view>
					<view class="action">
						<view>{{(repo.license && repo.license.spdx_id ? $_.toString(repo.license.spdx_id) + ' · ' : '') + $_.toString($NP.round($NP.divide(repo.size, 1024), 1)) + 'MB'}}</view>
					</view>
				</view>
				<!-- issues -->
				<view class="cu-item arrow">
					<view class="content">
						<text class="list-left-icon cu-avatar round iconfont iconissue" style="background-color: #37cb75;" />
						<text class="padding-left-sm">Issues</text>
					</view>
					<view class="action">
						<view>{{repo.open_issues}}</view>
					</view>
				</view>
				<!-- pull requests -->
				<view class="cu-item arrow">
					<view class="content">
						<text class="list-left-icon cu-avatar round iconfont iconpullrequest" style="background-color: #5756d5;" />
						<text class="padding-left-sm">Pull Requests</text>
					</view>
					<view class="action">
						<view>{{reposPullRequests.length}}</view>
					</view>
				</view>
				<u-gap />
				<!-- branches -->
				<view class="cu-item arrow">
					<view class="content">
						<text class="list-left-icon cu-avatar round iconfont icon24gf-branches" style="background-color: #434955;" />
						<text class="padding-left-sm">Branches</text>
					</view>
					<view class="action">
						<view>{{ (!$_.isEmpty(reposBranches) ? reposBranches[0].name + ' · ' : '') + $_.toString(reposBranches.length)}}</view>
					</view>
				</view>
				<!-- readme -->
				<view class="cu-item">
					<view class="content">
						<text class="list-left-icon cu-avatar round iconfont iconreadme" style="background-color: #028fff;" />
						<text class="padding-left-sm">Readme</text>
					</view>
					<view class="action">
						<view @tap="getReposReadme" class="iconfont iconrefresh" style="color:#028fff"></view>
					</view>
				</view>
			</view>
			<view class="order u-skeleton-rect bg-white">
				<text selectable v-html="reposReadme"></text>
			</view>

			<u-popup v-model="modelLangPercent" mode="top">
				<view class="cu-list menu card-menu margin-top">
					<view class="cu-item" v-for="item in reposLanguages" :key="item.lang">
						<view class="content">
							<text class="iconfont iconyuandianzhong margin-right-xs" :style="{'color':item.color}"></text>
							<text>{{item.lang}}</text>
						</view>
						<view class="action">
							<view>{{item.percent}}</view>
						</view>
					</view>
				</view>
			</u-popup>
		</scroll-view>
		<u-skeleton :loading="loading" :animation="true"></u-skeleton>
	</view>
</template>

<script>
	import { mapGetters } from 'vuex'
	import languageColors from '@/static/colors.json'
	export default {
		computed: {
			...mapGetters(['themeBgColor', 'darkMode'])
		},
		onReady() {
			uni.setNavigationBarTitle({
			    title: this.$t('Repos')
			})
			// navBar-bg-color
			uni.setNavigationBarColor({
			    frontColor: '#ffffff',
			    backgroundColor: this.themeBgColor,
			    animation: {
			        duration: 400,
			        timingFunc: 'easeIn'
			    }
			})
		},
		data() {
			return {
				loading: true,
				owner: '',
				repoName: '',
				modelLangPercent: false,
				numInfo: {},
				languageColor: '#ff7500',
				repo: {
					owner: {},
					license: {},
					size: 0
				},
				reposLanguages: [],
				reposPullRequests: [],
				reposBranches: [],
				reposReadme: ''
			}
		},
		onLoad(option) {
			this.owner = option.owner
			this.repoName = option.repo
			this.getRepos()
			this.listReposLanguages()
			this.listPullRequests()
			this.listBranches()
			this.getReposReadme()
		},
		methods: {
			imageError() {
				this.repo.owner.avatar_url = '/static/img/60x60.png'
			},
			async getRepos() {
				this.repo = await this.$minApi.getRepos(this.owner, this.repoName)
				this.numInfo = {
					watchers: this.repo.watchers,
					stars: this.repo.stargazers_count,
					forks: this.repo.forks
				}
			},
			async listReposLanguages() {
				const ret = await this.$minApi.listReposLanguages(this.owner, this.repoName)
				let sum = 0
				Object.values(ret).forEach(item => {
					sum = this.$NP.plus(sum, item)
				})
				let newRet = []
				for (let key in ret) {
					let lang = {
						'lang': key
					}
					lang['percent'] = this.$NP.round(this.$NP.times(this.$NP.divide(ret[key], sum), 100), 4) + '%'
					lang['color'] = languageColors[key].color
					newRet.push(lang)
				}
				this.reposLanguages = newRet
				if (!this.$_.isEmpty(newRet)) {
					this.languageColor = newRet[0].color
				}
			},
			async listPullRequests() {
				this.reposPullRequests = await this.$minApi.listPullRequests(this.owner, this.repoName)
			},
			async listBranches() {
				this.reposBranches = await this.$minApi.listBranches(this.owner, this.repoName)
			},
			async getReposReadme() {
				this.reposReadme = await this.$minApi.getReposReadme(this.owner, this.repoName)
				this.loading = false
			},
			clickLangPercent() {
				this.modelLangPercent = true
			},
			tapCode() {
				uni.navigateTo({
					url: `/pages/repos/code/code?owner=${this.owner}&repo=${this.repoName}&defaultBranch=${this.repo.default_branch}`
				})
			}
		}
	}
</script>
