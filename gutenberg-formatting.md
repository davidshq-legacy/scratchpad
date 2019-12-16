# Introduction
Gutenberg strips out specific HTML/CSS tags when content is pasted into the editor. This document is a scratch pad as I unravel this behavior.

Unless otherwise noted all file paths should have `/gutenberg/packages/` prepended to them.

# Paragraph
Location: `/block-library/src/paragraph`

The paragraph control is built on the RichText component (among others).

# RichText
Location: `/block-editor/src/components/rich-text`

RichText in turn depends on the `pasteHandler` provided by `@wordpress/blocks`.

In RichText we have a class `RichTextWrapper` which extends `Component`. Here we find an `onPaste` handler:
```
onPaste( { value, onChange, html, plainText, files, activeFormats } ) {
	const {
		onReplace,
		onSplit,
		tagName,
		canUserUseUnfilteredHTML,
		multiline,
		__unstableEmbedURLOnPaste,
	} = this.props;

	// Only process file if no HTML is present.
	// Note: a pasted file may have the URL as plain text.
	if ( files && files.length && ! html ) {
		const content = pasteHandler( {
			HTML: filePasteHandler( files ),
			mode: 'BLOCKS',
			tagName,
		} );

		// Allows us to ask for this information when we get a report.
		// eslint-disable-next-line no-console
		window.console.log( 'Received items:\n\n', files );

		if ( onReplace && isEmpty( value ) ) {
			onReplace( content );
		} else {
			this.onSplit( value, content );
		}

		return;
	}

	let mode = onReplace && onSplit ? 'AUTO' : 'INLINE';

	if (
		__unstableEmbedURLOnPaste &&
		isEmpty( value ) &&
		isURL( plainText.trim() )
	) {
		mode = 'BLOCKS';
	}

	const content = pasteHandler( {
		HTML: html,
		plainText,
		mode,
		tagName,
		canUserUseUnfilteredHTML,
	} );

	if ( typeof content === 'string' ) {
		let valueToInsert = create( { html: content } );

		// If there are active formats, merge them with the pasted formats.
		if ( activeFormats.length ) {
			let index = valueToInsert.formats.length;

			while ( index-- ) {
				valueToInsert.formats[ index ] = [
					...activeFormats,
					...( valueToInsert.formats[ index ] || [] ),
				];
			}
		}

		// If the content should be multiline, we should process text
		// separated by a line break as separate lines.
		if ( multiline ) {
			valueToInsert = replace( valueToInsert, /\n+/g, LINE_SEPARATOR );
		}

		onChange( insert( value, valueToInsert ) );
	} else if ( content.length > 0 ) {
		if ( onReplace && isEmpty( value ) ) {
			onReplace( content );
		} else {
			this.onSplit( value, content );
		}
	}
  ```
  
  This `onPaste` handler calls `pasteHandler` and passes to it `HTML: html, plainText, mode, tagName, canUserUseUnfilteredHTML`.

  
  
