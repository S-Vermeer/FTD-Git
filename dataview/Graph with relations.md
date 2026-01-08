```dataviewjs
const blockEl = this.container; 
if (!window.vis) { 
	const s = document.createElement('script'); 
	s.src = 'https://unpkg.com/vis-network@9.1.2/dist/vis-network.min.js';
	s.onload = () => draw(blockEl); 
	document.head.appendChild(s); 
} else { 
	draw(this.container); 
} 

function draw(parent) { 
	const pages = dv.pages('').where(p => p.type); 
	const nodes = [], edges = [], idMap = new Map(); let next = 1; 
	const edgeInfoMap = new Map();
	for (const p of pages) { 
		const id = next++; 
		idMap.set(p.file.path, id); 
		nodes.push({ id, label: p.name || p.file.name, shape: 'ellipse', color: '#4A90E2' }); 
		} 
		for (const p of pages) { 
			if (!Array.isArray(p.relations)) continue; 
			const from = idMap.get(p.file.path); 
			for (const r of p.relations) { 
				const targetPage = dv.page(r.target); 
				if (!targetPage) continue; 
				const to = idMap.get(targetPage.file.path); 
				if (to === undefined) continue; 
				const a = Math.min(from, to); 
				const b = Math.max(from, to); 
				const key = `${a}-${b}-${r.kind}`; 
				if (!edgeInfoMap.has(key)) { 
					edgeInfoMap.set(key, { from, to, label: r.kind, count: 1 }); 
				} else { 
					const info = edgeInfoMap.get(key); 
					info.count += 1; 
				} 
			} 
		} 
		for (const info of edgeInfoMap.values()) { 
			if (info.count === 2) { 
				edges.push({ from: info.from, to: info.to, label: info.label, arrows: { to: true, from: true }, color: { color: '#888888' } }); 
			} else { 
				edges.push({ from: info.from, to: info.to, label: info.label, arrows: 'to', color: { color: '#888888' } }); 
			} 
		} 
		const container = document.createElement('div');
		container.style.width = '100%'; 
		container.style.height = '600px'; 
		parent.appendChild(container); 
		const data = { nodes: new vis.DataSet(nodes), edges: new vis.DataSet(edges) }; 
		const opts = { 
			physics: { stabilization: false, barnesHut: { gravitationalConstant: -8000 } }, 
			interaction: { hover: true, navigationButtons: true }, 
			edges: { font: { align: 'middle' } } 
		}; 
		const network = new vis.Network(container, data, opts); 
		
		const exportSvgBtn = document.createElement('button')
		exportSvgBtn.textContent = 'Export SVG'
		exportSvgBtn.style.marginLeft = '8px'
		exportSvgBtn.onclick = () => {  

			const canvas = container.querySelector('canvas');
			if (!canvas) {
			    alert('Canvas not found – cannot export');
			    return;
			}
			  // Get a data‑URL (PNG) from the canvas
			const imgData = canvas.toDataURL('image/png');
			
			  // Trigger a download
			const a = document.createElement('a');
			a.href = imgData;
			a.download = 'knowledge-graph.png';
			a.click();


	    // Clean up the object URL
	    setTimeout(() => URL.revokeObjectURL(url), 100)
}
parent.appendChild(exportSvgBtn)
}
```