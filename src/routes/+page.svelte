<script lang="ts">
	import JSZip from 'jszip'
	import FileSaver from 'file-saver'

	let files: any
	let downloads: { href: string; download: string }[] = []

	$: newFiles(files?.[0])

	function newFiles(file: File) {
		if (!file) return

		const reader = new FileReader()
		reader.readAsText(file, 'UTF-8')
		reader.onload = function (e) {
			const mails = e.target!.result?.toString()!
			const mailByDomain: { [domain: string]: string[] } = {}
			for (const mail of mails.split('\n')) {
				const domain = mail?.split('@')[1]?.trim()
				if (!domain) continue

				if (!mailByDomain[domain]) mailByDomain[domain] = []
				mailByDomain[domain].push(mail.trim())
			}
			downloads = Object.entries(mailByDomain).map(([domain, mails]) => ({
				href: `data:text/plain;charset=utf-8,${encodeURIComponent(mails.join('\n'))}`,
				download: `${domain}.txt`,
			}))

			const zip = new JSZip()
			Object.entries(mailByDomain).forEach(([domain, mails]) =>
				zip.file(`${domain}.txt`, mails.join('\n')),
			)

			zip.generateAsync({ type: 'base64' }).then((content) => {
				downloads = [
					{ href: 'data:application/zip;base64,' + content, download: 'mails.zip' },
					...downloads,
				]
			})
		}
	}
</script>

<input type="file" bind:files />

{#each downloads as { href, download }}
	<a {href} {download}>{download}</a>
{/each}

<style>
	a {
		display: block;
	}
</style>
