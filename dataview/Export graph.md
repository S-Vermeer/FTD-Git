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

async function draw(parent) { 
	const pages = dv.pages('').where(p => p.type); 
	const nodes = [], edges = [], idMap = new Map(); 
	let next = 1; const edgeInfoMap = new Map(); 
	
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
				edgeInfoMap.get(key).count += 1; } 
			} 
		} 
		for (const info of edgeInfoMap.values()) { 
			if (info.count === 2) { 
				edges.push({ from: info.from, to: info.to, label: info.label, arrows: { to: true, from: true }, color: { color: '#888888' } }); 
			} else { 
				edges.push({ from: info.from, to: info.to, label: info.label, arrows: 'to', color: { color: '#888888' } }); 
			} 
		}
		
		const canvasDiv = document.createElement('div'); 
		canvasDiv.style.width = '100%'; 
		canvasDiv.style.height = '600px'; 
		parent.appendChild(canvasDiv); 
		const data = { nodes: new vis.DataSet(nodes), edges: new vis.DataSet(edges) }; 
		const opts = { physics: { stabilization: false, barnesHut: { gravitationalConstant: -8000 } }, interaction: { hover: true, navigationButtons: true }, edges: { font: { align: 'middle' } } }; 
		const network = new vis.Network(canvasDiv, data, opts);
		

		await new Promise(resolve => {
			const d3script = document.createElement('script');
			d3script.src = 'https://cdn.jsdelivr.net/npm/d3@7';
			d3script.onload = resolve;
			document.head.appendChild(d3script);
		});
		
		const svgNS = 'http://www.w3.org/2000/svg'; 
		const svg = document.createElementNS(svgNS, 'svg');
		svg.setAttribute('width', 1200); 
		svg.setAttribute('height', 800); 
		svg.style.display = 'none';
		parent.appendChild(svg); 
	//	const d3script = document.createElement('script');
	//	d3script.src = 'https://cdn.jsdelivr.net/npm/d3@7';
	//	d3script.onload = resolve;
	//	document.head.appendChild(d3script);
//		});

		const simulation = d3.forceSimulation(nodes) 
		.force('link', d3.forceLink(edges).id(d => d.id).distance(150))
		.force('charge', d3.forceManyBody().strength(-500)) 
		.force('center', d3.forceCenter(600, 400)); 
		
		for (let i = 0; i < 300; ++i) simulation.tick();
		
		const edgeGroup = document.createElementNS(svgNS, 'g');
		edgeGroup.setAttribute('stroke', '#888'); 
		edgeGroup.setAttribute('stroke-width', '2'); 
		svg.appendChild(edgeGroup); 
		edges.forEach(e => { 
			const line = document.createElementNS(svgNS, 'line');
			const source = nodes.find(n => n.id === e.from); 
			const target = nodes.find(n => n.id === e.to); 
			line.setAttribute('x1', source.x); 
			line.setAttribute('y1', source.y); 
			line.setAttribute('x2', target.x); 
			line.setAttribute('y2', target.y); 
			edgeGroup.appendChild(line); 
		}); 
		
		const nodeGroup = document.createElementNS(svgNS, 'g');
		nodeGroup.setAttribute('fill', '#4A90E2'); 
		svg.appendChild(nodeGroup); 
		nodes.forEach(n => { 
			const circle = document.createElementNS(svgNS, 'circle');
			circle.setAttribute('cx', n.x); circle.setAttribute('cy', n.y);
			circle.setAttribute('r', 30); nodeGroup.appendChild(circle); 
		});
		
		const labelGroup = document.createElementNS(svgNS, 'g');
		labelGroup.setAttribute('font-family', 'sans-serif');
		labelGroup.setAttribute('font-size', '12');
		labelGroup.setAttribute('fill', '#000'); 
		svg.appendChild(labelGroup); 
		nodes.forEach(n => { 
			const txt = document.createElementNS(svgNS, 'text');
			txt.setAttribute('x', n.x); 
			txt.setAttribute('y', n.y + 4);
			txt.setAttribute('text-anchor', 'middle'); 
			txt.textContent = n.label; 
			labelGroup.appendChild(txt); 
		});
		
		const pngBtn = document.createElement('button');
		pngBtn.textContent = 'Export PNG';
		pngBtn.style.marginTop = '8px';
		pngBtn.onclick = () => {
			const dataURL = network.canvas.frame.canvas.toDataURL('image/png');
			const a = document.createElement('a');
			a.href = dataURL; 
			a.download = 'knowledge-graph.png'; 
			a.click(); 
		};
		parent.appendChild(pngBtn);
		
		const svgBtn = document.createElement('button');
		svgBtn.textContent = 'Export SVG';
		svgBtn.style.marginLeft = '8px';
		svgBtn.onclick = () => { 
			const serializer = new XMLSerializer();
			const svgText = serializer.serializeToString(svg);
			const blob = new Blob([svgText], { type: 'image/svg+xml;charset=utf-8' }); 
			const url = URL.createObjectURL(blob); 
			const a = document.createElement('a'); 
			a.href = url;
			a.download = 'knowledge-graph.svg'; 
			a.click(); 
			setTimeout(() => URL.revokeObjectURL(url), 100);
		};
		parent.appendChild(svgBtn);
		
		const jsonBtn = document.createElement('button');
		jsonBtn.textContent = 'Export JSON';
		jsonBtn.style.marginLeft = '8px';
		jsonBtn.onclick = () => {
			const json = { nodes: data.nodes.get(), edges: data.edges.get() }; 
			const blob = new Blob([JSON.stringify(json, null, 2)], { type: 'application/json' });
			const url = URL.createObjectURL(blob);
			const a = document.createElement('a');
			a.href = url;
			a.download = 'knowledge-graph.json';
			a.click();
			setTimeout(() => URL.revokeObjectURL(url), 100);
		};
		parent.appendChild(jsonBtn);
	}
```