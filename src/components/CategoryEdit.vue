<template>
	<div class="col s12 m6">
		<div>
			<div class="page-subtitle">
				<h4>Редактировать</h4>
			</div>

			<form @submit.prevent="submitHandler">
				<div class="input-field" >
					<select ref="select" v-model="current">
						<option
							v-for="c of categories"
							:key = "c.id"
							:value = "c.id"
						>{{c.title}}
						</option>
					</select>
					<label>Выберите категорию</label>
				</div>

				<div class="input-field">
					<input
							id="name"
							type="text"
							v-model="title"
							:class="{invalid: ($v.title.$dirty && !$v.title.required)}"
					>
					<label for="name">Название</label>
					<span
						v-if="$v.title.$dirty && !$v.title.required" 
						class="helper-text invalid"
					>
						Введите название категории
					</span>
				</div>

				<div class="input-field">
					<input
							id="limit"
							type="number"
							v-model.number="limit"
							:class="{invalid: ($v.title.$dirty && !$v.title.minValue)}"
					>
					<label for="limit">Лимит</label>
					<span
						v-if="$v.title.$dirty && !$v.title.minValue" 
						class="helper-text invalid"
					>
						Минимальная величина
					</span>
				</div>

				<button class="btn waves-effect waves-light" type="submit">
					Обновить
					<i class="material-icons right">send</i>
				</button>
			</form>
		</div>
	</div>
</template>

<script>
	import {required, minValue} from 'vuelidate/lib/validators'

	export default {
		props: {
			categories: {
				type: Array,
				required: true
			}
		},
		data: () => ({
			select: null,
			title: '',
			limit: 100,
			current: null
		}),
		validations: {
			title: {required},
			limit: {minValue: minValue(1)}
		},
		watch: {
			current(catId) {
				const {title, limit} = this.categories.find(c => c.id === catId)
				this.title = title
				this.limit = limit
			}
		},
		created() {
			const {id, title, limit} = this.categories[0]
			this.current = id
			this.title = title
			this.limit = limit
		},
		methods: {
			async submitHandler() {
				if (this.$v.$invalid) {
					this.$v.$touch()
					return
				}

				try {
					const categoryData = {
						id: this.current,
						title: this.title,
						limit: this.limit
					}
					await this.$store.dispatch('updateCategory', categoryData)
					this.$message('Категория обновлена')
					this.$emit('updated', categoryData)
				} catch (e){
					console.log(e)
				}
			}
		},
		mounted() {
			this.select = window.M.FormSelect.init(this.$refs.select)
			window.M.updateTextFields()
		},
		destroyed() {
			if (this.select && this.select.destroy) {
				this.select.destroy()
			}
		}
	}
</script>