# deepFilterHTML
Location: `/blocks/src/api/raw-handling/utils.js`

At its core `deepFilterHTML` uses `deepFilterNodeList`

# deepFilterNodeList
Location: `/blocks/src/api/raw-handling/utils.js`
```
/**
 * Given node filters, deeply filters and mutates a NodeList.
 *
 * @param {NodeList} nodeList The nodeList to filter.
 * @param {Array}    filters  An array of functions that can mutate with the provided node.
 * @param {Document} doc      The document of the nodeList.
 * @param {Object}   schema   The schema to use.
 */
export function deepFilterNodeList( nodeList, filters, doc, schema ) {
	Array.from( nodeList ).forEach( ( node ) => {
		deepFilterNodeList( node.childNodes, filters, doc, schema );

		filters.forEach( ( item ) => {
			// Make sure the node is still attached to the document.
			if ( ! doc.contains( node ) ) {
				return;
			}

			item( node, doc, schema );
		} );
	} );
}
```

# removeInvalidHTML
Location: `/blocks/src/api/raw-handling/utils.js`
At its core `removeInvalidHTML` uses `cleanNodeList`.
