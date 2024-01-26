# dialog-error

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun SearchSaleNoCodeDialog(
    context: Context,
    onDismiss: () -> Unit,
    viewModel: MainViewModel,
    isKeyboardVisible: State<Boolean>
) {

    var searchText by remember { mutableStateOf("") }
    var searchResults2 by remember {
        mutableStateOf<List<ProductSaleNoCode>>(emptyList())
    }
    var searchResults by remember { mutableStateOf<List<ProductSale>>(emptyList()) }
    val coroutineScope = rememberCoroutineScope()
    var searchJob by remember { mutableStateOf<Job?>(null) }
    val cart2Items = viewModel.cartProducts2.observeAsState().value
    val cart3Items = viewModel.cartProducts3.observeAsState().value
    val keyboardController = LocalSoftwareKeyboardController.current
    val focusRequester = remember { FocusRequester() }
    val focusManager = LocalFocusManager.current

    AlertDialog(
        containerColor = text_field_search_bg.copy(alpha = 0.85f),
        onDismissRequest = {
            hideKeyboard(context = context)
            onDismiss()
        },
        title = {
            Row(Modifier.fillMaxWidth(), verticalAlignment = Alignment.CenterVertically) {
                Spacer(modifier = Modifier.width(MaterialTheme.dimens.keyboardPadding * 2))
                Text(
                    stringResource(id = R.string.SearchProduct),
                    color = item_bg_color,
                    fontWeight = FontWeight.SemiBold,
                    fontSize = MaterialTheme.typography.headlineLarge.fontSize,
                )
                Spacer(modifier = Modifier.width(MaterialTheme.dimens.padding16))
                Icon(
                    painter = painterResource(id = R.drawable.ic_quantity),
                    contentDescription = null,
                    tint = item_bg_color,
                    modifier = Modifier.size(MaterialTheme.dimens.debtsCardIconSize)
                )
            }
        },
        text = {
            Column(modifier = Modifier.background(color = Color.Transparent)) {
                TextField(
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(MaterialTheme.dimens.padding16 * 2.9f)
                        .clip(RoundedCornerShape(32.dp))
                        .focusRequester(focusRequester),
                    value = searchText,
                    onValueChange = { newText ->
                        // Check if the newText is not just leading whitespace added to the searchText
                        if (newText.isNotBlank() || newText.isEmpty()) {
                            // Only update searchText if newText has non-whitespace characters or is empty
                            searchText = newText.trimStart()
                        }

                        searchText = newText
                        if (searchText.isEmpty()) {
                            searchResults = emptyList()
                            searchResults2 = emptyList()
                        }
                        // searchJob?.cancel() // Cancel the previous search job if it's still active
                        searchResults = emptyList() // Clear existing results immediately
                        searchResults2 = emptyList()
                        searchJob = coroutineScope.launch {
                            delay(200)  // Debounce by 200ms
                            if (newText == searchText && newText.isNotEmpty()) {
                                searchSaleProductsFromSale(
                                    context = context,
                                    query = newText,
                                    onResult = { items ->
                                        searchResults2 = items
                                    },
                                    onError = { error ->
                                        Log.e("SearchDialog", error)
                                    }
                                )
                            } else {
                                searchResults = emptyList()
                                searchResults2 = emptyList()
                            }
                        }
                    },
                    singleLine = true,
                    textStyle = TextStyle(
                        fontSize = MaterialTheme.typography.labelMedium.fontSize,
                        color = item_bg_color
                    ),
                    colors = TextFieldDefaults.outlinedTextFieldColors(
                        focusedBorderColor = Color.Transparent,
                        unfocusedBorderColor = Color.Transparent,
                        cursorColor = item_bg_color,
                        containerColor = item_bg_color_lighter2
                    )
                )
                LaunchedEffect(Unit) {
                    focusRequester.requestFocus()
                }
                LazyColumn(
                    modifier = Modifier
                        .fillMaxWidth()
                        .heightIn(max = MaterialTheme.dimens.debtsCardIconSize * 10)
                ) {
                    val itemsToDisplay = searchResults2.filter { result ->
                        cart3Items!!.none { cartItems ->
                            cartItems.id == result.id // Assuming 'id' uniquely identifies the item
                        }
                    }
                    items(itemsToDisplay) { result ->
                        Text(
                            text = result.title,
                            color = item_bg_color,
                            fontSize = MaterialTheme.typography.labelLarge.fontSize,
                            fontWeight = FontWeight.SemiBold,
                            modifier = Modifier
                                .fillMaxWidth()
                                .padding(MaterialTheme.dimens.padding16)
                                .clickable {
                                    // all this methods is not working
                                    focusManager.clearFocus()
                                    hideKeyboard(context)
                                    keyboardController?.hide()
                                        val codeModel3Item = CodeModel3(
                                        id = result.id,
                                        title = result.title,
                                        price = result.price,
                                        main_image = result.mainImage.toString(),
                                        max_quantity = result.max_quantity,
                                        expiration_date = result.expirationDate ?: "",
                                        userSelectedQuantity = 1 // The default starting weight
                                    )
                                         viewModel.addToCart3(listOf(codeModel3Item))
                                         viewModel.isSearchNoCodeDialogOpen(false)
                                }

                        )
                    }
                }
            }
        },
        confirmButton = {
            Button(colors = ButtonDefaults.buttonColors(
                containerColor = item_bg_color_lighter2
            ), shape = CircleShape, onClick = {
                    focusManager.clearFocus(force = true)
                    hideKeyboard(context)
                    keyboardController?.hide()
            // click ishlavoti
            }) {
                Text(
                    text = stringResource(id = R.string.Close),
                    color = item_bg_color,
                    fontSize = MaterialTheme.typography.headlineSmall.fontSize,
                    fontFamily = FontFamily.Monospace,
                    fontWeight = FontWeight.SemiBold, // Makes the font bold
                    textAlign = TextAlign.Center // Centers the text
                )
            }
        }
    )
}
