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

# Paste Handler
Location: `/blocks/src/api/raw-handling/paste-handler.js`

The pasteHandler takes five parameters:
- `HTML` (default '')
- `plainText` (default '')
- `mode` (default 'AUTO')
- `tagName`
- `canUserUseUnfilteredHTML` (default false)

It then performs the following actions on the passed content:

- Strip meta tags
- Strip window markers (appears to be anything from `<html>` until `<body>` and from `</body>` to `</html>`.
- Checks if blocks are being passed, if so, handles them as blocks.
- Normalize unicode to use composed characters.
- Parses markdown, if present.
  - During this process `mode` may change from `AUTO` to `INLINE`. This is not important for our purposes.
- If there are shortcodes in the pasted content any that are recognized are converted using the `shortcodeConverter()` function - again, not important for our purposes.
- Under certain conditions the pasted content is run through `filteredInlineHTML`.
- We set `rawTransforms` equal to `getRawTransformations()`, `phrasingContentSchema` to `getPhrasingContentScheme()`, and `blockContentSchema` to `getBlockContentSchema()`.
- Note that `rawTransforms` and `phrasingContentSchema` are then passed as parameters to `blockContentSchema`.
- The code handles shortcodes.
- Now a number of filters are defined: `googleDocsUIDRemover`, `msListConverter`, `headRemover`, `listReducer`, `imageCorrector`, `phrasingContentReducer`, `specialCommentConverter`, `commentRemover`, `figureContentReducer`, `blockquoteNormaliser`. 
- If `canUserUseUnfilteredHTML` is false then `iframeRemover` is added to the front of the `filters` array above.
- Define constant `schema` which takes as parameters `blockContentSchema` and `phrasingContentSchema`.
- The `piece` of content is run through specific filters: `deepFilterHTML`, `removeInvalidHTML`, `normaliseBlocks`, and `deepFilterHTML` again - this time with specific formats passed: `htmlFormattingRemover`, `brRemover`, `emptyParagraphRemover`.
- Some information is logged to the console and then `htmlToBlocks` is returned with `{ html: piece, rawTransforms }`.
- One final special case is handled and content returned.
