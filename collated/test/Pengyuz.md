# Pengyuz
###### /java/seedu/address/logic/commands/DeleteCommandTest.java
``` java
/**
 * Contains integration tests (interaction with the Model) and unit tests for {@code DeleteCommand}.
 */
public class DeleteCommandTest {

    private static final ReadOnlyPerson DUPLICATE = new PersonBuilder().withName("Alice Pauline")
            .withAddress("124, Jurong West Ave 7, #08-112").withEmail("alicee@example.com")
            .withPhone("85333333")
            .withTags("workmate").build();
    private Model model = new ModelManager(getTypicalAddressBook(), new UserPrefs());
    private ArrayList<Index> personsToDelete1 = new ArrayList<>();

    @Test
    public void excute_duplicatePerson_sucess() throws Exception {

        String duplicate = "Alice Pauline";
        model.addPerson(DUPLICATE);

        List<String> duplicatePerson = Arrays.asList(duplicate);
        NameContainsKeywordsPredicate updatedpredicate = new NameContainsKeywordsPredicate(duplicatePerson);

        DeleteCommand deleteCommand = prepareCommand(duplicate);

        String expectedMessage = "Duplicate persons exist, please choose one to delete.";

        ModelManager expectedModel = new ModelManager(model.getAddressBook(), new UserPrefs());

        expectedModel.updateFilteredPersonList(updatedpredicate);

        assertCommandSuccess(deleteCommand, model, expectedMessage, expectedModel);

        model.deletePerson(DUPLICATE);

    }


    @Test
    public void execute_validIndexUnfilteredList_success() throws Exception {
        ReadOnlyPerson personToDelete = model.getFilteredPersonList().get(INDEX_FIRST_PERSON.getZeroBased());

        personsToDelete1.add(INDEX_FIRST_PERSON);

        DeleteCommand deleteCommand1 = prepareCommand(personsToDelete1);

        String expectedMessage1 = DeleteCommand.MESSAGE_DELETE_PERSON_SUCCESS;

        ModelManager expectedModel1 = new ModelManager(model.getAddressBook(), new UserPrefs());

        expectedModel1.deletePerson(personToDelete);

        assertCommandSuccess(deleteCommand1, model, expectedMessage1, expectedModel1);
    }

    @Test
    public void execute_validIndexUnfilteredList_success2() throws Exception {
        ReadOnlyPerson personToDelete = model.getFilteredPersonList().get(INDEX_FIRST_PERSON.getZeroBased());
        ReadOnlyPerson secondToDelete = model.getFilteredPersonList().get(INDEX_SECOND_PERSON.getZeroBased());

        personsToDelete1.clear();
        personsToDelete1.add(INDEX_FIRST_PERSON);
        personsToDelete1.add(INDEX_SECOND_PERSON);

        DeleteCommand deleteCommand1 = prepareCommand(personsToDelete1);

        String expectedMessage1 = DeleteCommand.MESSAGE_DELETE_PERSON_SUCCESS;

        ModelManager expectedModel1 = new ModelManager(model.getAddressBook(), new UserPrefs());

        expectedModel1.deletePerson(personToDelete);
        expectedModel1.deletePerson(secondToDelete);

        assertCommandSuccess(deleteCommand1, model, expectedMessage1, expectedModel1);
    }

    @Test
    public void execute_validNameUnfilteredList_success() throws Exception {
        ReadOnlyPerson personToDelete = model.getFilteredPersonList().get(INDEX_FIRST_PERSON.getZeroBased());

        personsToDelete1.clear();
        String deleteName = personToDelete.getName().fullName;

        DeleteCommand deleteCommand1 = prepareCommand(deleteName);

        String expectedMessage1 = DeleteCommand.MESSAGE_DELETE_PERSON_SUCCESS;

        ModelManager expectedModel1 = new ModelManager(model.getAddressBook(), new UserPrefs());

        expectedModel1.deletePerson(personToDelete);

        assertCommandSuccess(deleteCommand1, model, expectedMessage1, expectedModel1);
    }


    @Test
    public void execute_invalidIndexUnfilteredList_throwsCommandException() throws Exception {
        Index outOfBoundIndex = Index.fromOneBased(model.getFilteredPersonList().size() + 1);
        personsToDelete1.clear();
        personsToDelete1.add(outOfBoundIndex);
        DeleteCommand deleteCommand = prepareCommand(personsToDelete1);

        assertCommandFailure(deleteCommand, model, Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
    }

    @Test
    public void execute_validIndexFilteredList_success() throws Exception {
        showFirstPersonOnly(model);

        ReadOnlyPerson personToDelete = model.getFilteredPersonList().get(INDEX_FIRST_PERSON.getZeroBased());
        personsToDelete1.clear();
        personsToDelete1.add(INDEX_FIRST_PERSON);
        DeleteCommand deleteCommand = prepareCommand(personsToDelete1);

        String expectedMessage = String.format(DeleteCommand.MESSAGE_DELETE_PERSON_SUCCESS, personToDelete);

        Model expectedModel = new ModelManager(model.getAddressBook(), new UserPrefs());
        expectedModel.deletePerson(personToDelete);
        showNoPerson(expectedModel);

        assertCommandSuccess(deleteCommand, model, expectedMessage, expectedModel);
    }

    @Test
    public void execute_validNameFilteredList_success() throws Exception {
        showFirstPersonOnly(model);

        ReadOnlyPerson personToDelete = model.getFilteredPersonList().get(INDEX_FIRST_PERSON.getZeroBased());

        personsToDelete1.clear();
        String deleteName = personToDelete.getName().fullName;
        DeleteCommand deleteCommand = prepareCommand(deleteName);

        String expectedMessage = String.format(DeleteCommand.MESSAGE_DELETE_PERSON_SUCCESS, personToDelete);

        Model expectedModel = new ModelManager(model.getAddressBook(), new UserPrefs());
        expectedModel.deletePerson(personToDelete);
        showNoPerson(expectedModel);

        assertCommandSuccess(deleteCommand, model, expectedMessage, expectedModel);
    }

    @Test
    public void execute_invalidIndexFilteredList_throwsCommandException() {
        showFirstPersonOnly(model);

        Index outOfBoundIndex = INDEX_SECOND_PERSON;
        personsToDelete1.clear();
        personsToDelete1.add(INDEX_SECOND_PERSON);
        // ensures that outOfBoundIndex is still in bounds of address book list
        assertTrue(outOfBoundIndex.getZeroBased() < model.getAddressBook().getPersonList().size());

        DeleteCommand deleteCommand = prepareCommand(personsToDelete1);

        assertCommandFailure(deleteCommand, model, Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
    }

    @Test
    public void execute_invalidNameFilteredList_throwsCommandException() {
        model = new ModelManager(getTypicalAddressBook(), new UserPrefs());
        Index outOfBoundIndex = INDEX_SECOND_PERSON;
        personsToDelete1.clear();
        ReadOnlyPerson personToDelete = model.getFilteredPersonList().get(INDEX_SECOND_PERSON.getZeroBased());
        String name = personToDelete.getName().fullName;

        showFirstPersonOnly(model);


        // ensures that outOfBoundIndex is still in bounds of address book list
        assertTrue(outOfBoundIndex.getZeroBased() < model.getAddressBook().getPersonList().size());

        DeleteCommand deleteCommand = prepareCommand(name);

        assertCommandFailure(deleteCommand, model, Messages.MESSAGE_INVALID_PERSON_DISPLAYED_INDEX);
    }


    @Test
    public void equals() {
        ArrayList<Index> first = new ArrayList<>();
        ArrayList<Index> second = new ArrayList<>();
        first.add(INDEX_FIRST_PERSON);
        second.add(INDEX_SECOND_PERSON);
        DeleteCommand deleteFirstCommand = new DeleteCommand(first);
        DeleteCommand deleteSecondCommand = new DeleteCommand(second);

        // same object -> returns true
        assertTrue(deleteFirstCommand.equals(deleteFirstCommand));

        // same values -> returns true
        DeleteCommand deleteFirstCommandCopy = new DeleteCommand(first);
        assertTrue(deleteFirstCommand.equals(deleteFirstCommandCopy));

        // different types -> returns false
        assertFalse(deleteFirstCommand.equals(1));

        // null -> returns false
        assertFalse(deleteFirstCommand.equals(null));

        // different person -> returns false
        assertFalse(deleteFirstCommand.equals(deleteSecondCommand));
    }

    /**
     * Returns a {@code DeleteCommand} with the parameter {@code index}.
     */
    private DeleteCommand prepareCommand(ArrayList<Index> indexes) {

        DeleteCommand deleteCommand = new DeleteCommand(indexes);
        deleteCommand.setData(model, new CommandHistory(), new UndoRedoStack());
        return deleteCommand;
    }

    private DeleteCommand prepareCommand(String name) {
        DeleteCommand deleteCommand = new DeleteCommand(name);
        deleteCommand.setData(model, new CommandHistory(), new UndoRedoStack());
        return deleteCommand;
    }


    /**
     * Updates {@code model}'s filtered list to show no one.
     */
    private void showNoPerson(Model model) {
        model.updateFilteredPersonList(p -> false);

        assert model.getFilteredPersonList().isEmpty();
    }

}
```
###### /java/seedu/address/logic/commands/HelpCommandTest.java
``` java

public class HelpCommandTest {
    @Rule
    public final EventsCollectorRule eventsCollectorRule = new EventsCollectorRule();

    @Test
    public void execute_help_success() {
        CommandResult result = new HelpCommand().execute();
        assertEquals(SHOWING_HELP_MESSAGE, result.feedbackToUser);
        assertTrue(eventsCollectorRule.eventsCollector.getMostRecent() instanceof ShowHelpRequestEvent);
        assertTrue(eventsCollectorRule.eventsCollector.getSize() == 1);
    }

    @Test
    public void excutesuccess() {
        CommandResult result = new HelpCommand("add").execute();
        assertEquals(AddCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("clear").execute();
        assertEquals(ClearCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("delete").execute();
        assertEquals(DeleteCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("edit").execute();
        assertEquals(EditCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("exit").execute();
        assertEquals(ExitCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("find").execute();
        assertEquals(FindCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("history").execute();
        assertEquals(HistoryCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("list").execute();
        assertEquals(ListCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("redo").execute();
        assertEquals(RedoCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("select").execute();
        assertEquals(SelectCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("sort").execute();
        assertEquals(SortCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("tagadd").execute();
        assertEquals(TagAddCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("tagremove").execute();
        assertEquals(TagRemoveCommand.MESSAGE_USAGE, result.feedbackToUser);

        result = new HelpCommand("undo").execute();
        assertEquals(UndoCommand.MESSAGE_USAGE, result.feedbackToUser);
    }
}
```
###### /java/seedu/address/logic/parser/AddressBookParserTest.java
``` java
public class AddressBookParserTest {
    @Rule
    public ExpectedException thrown = ExpectedException.none();

    private final AddressBookParser parser = new AddressBookParser();

    @Test
    public void parseCommand_add() throws Exception {
        Person person = new PersonBuilder().build();
        AddCommand command = (AddCommand) parser.parseCommand(PersonUtil.getAddCommand(person));
        assertEquals(new AddCommand(person), command);
    }

    @Test
    public void parseCommand_create() throws Exception {
        Person person = new PersonBuilder().build();
        AddCommand command = (AddCommand) parser.parseCommand(PersonUtil.getCreateCommand(person));
        assertEquals(new AddCommand(person), command);
    }

    @Test
    public void parseCommand_put() throws Exception {
        Person person = new PersonBuilder().build();
        AddCommand command = (AddCommand) parser.parseCommand(PersonUtil.getPutCommand(person));
        assertEquals(new AddCommand(person), command);
    }

    @Test
    public void parseCommand_clear() throws Exception {
        assertTrue(parser.parseCommand(ClearCommand.COMMAND_WORD) instanceof ClearCommand);
        assertTrue(parser.parseCommand(ClearCommand.COMMAND_WORD + " 3") instanceof ClearCommand);
    }

    @Test
    public void parseCommand_delete() throws Exception {
        ArrayList<Index> todelete = new ArrayList<>();
        todelete.add(INDEX_FIRST_PERSON);
        DeleteCommand command = (DeleteCommand) parser.parseCommand(
                DeleteCommand.COMMAND_WORD + " I/" + INDEX_FIRST_PERSON.getOneBased());
        assertEquals(new DeleteCommand(todelete), command);
    }

    @Test
    public void parseCommand_remove() throws Exception {
        ArrayList<Index> todelete = new ArrayList<>();
        todelete.add(INDEX_FIRST_PERSON);
        DeleteCommand command = (DeleteCommand) parser.parseCommand(
                DeleteCommand.COMMAND_WORD_2 + " I/" + INDEX_FIRST_PERSON.getOneBased());
        assertEquals(new DeleteCommand(todelete), command);
    }

    @Test
    public void parseCommand_minus() throws Exception {
        ArrayList<Index> todelete = new ArrayList<>();
        todelete.add(INDEX_FIRST_PERSON);
        DeleteCommand command = (DeleteCommand) parser.parseCommand(
                DeleteCommand.COMMAND_WORD_3 + " I/" + INDEX_FIRST_PERSON.getOneBased());
        assertEquals(new DeleteCommand(todelete), command);
    }

    @Test
    public void parseCommand_edit() throws Exception {
        Person person = new PersonBuilder().build();
        EditPersonDescriptor descriptor = new EditPersonDescriptorBuilder(person).build();
        EditCommand command = (EditCommand) parser.parseCommand(EditCommand.COMMAND_WORD + " "
                + INDEX_FIRST_PERSON.getOneBased() + " " + PersonUtil.getPersonDetails(person));
        assertEquals(new EditCommand(INDEX_FIRST_PERSON, descriptor), command);
    }

    @Test
    public void parseCommand_modify() throws Exception {
        Person person = new PersonBuilder().build();
        EditPersonDescriptor descriptor = new EditPersonDescriptorBuilder(person).build();
        EditCommand command = (EditCommand) parser.parseCommand(EditCommand.COMMAND_WORD_3 + " "
                + INDEX_FIRST_PERSON.getOneBased() + " " + PersonUtil.getPersonDetails(person));
        assertEquals(new EditCommand(INDEX_FIRST_PERSON, descriptor), command);
    }

    @Test
    public void parseCommand_update() throws Exception {
        Person person = new PersonBuilder().build();
        EditPersonDescriptor descriptor = new EditPersonDescriptorBuilder(person).build();
        EditCommand command = (EditCommand) parser.parseCommand(EditCommand.COMMAND_WORD_2 + " "
                + INDEX_FIRST_PERSON.getOneBased() + " " + PersonUtil.getPersonDetails(person));
        assertEquals(new EditCommand(INDEX_FIRST_PERSON, descriptor), command);
    }

    @Test
    public void parseCommandTagAdd() throws Exception {
        Person person = new PersonBuilder().build();
        ArrayList<Index> singlePersonIndexList = new ArrayList<>();
        singlePersonIndexList.add(INDEX_FIRST_PERSON);
        Set<Tag> tagSet = new HashSet<>();
        tagSet.add(new Tag("friend"));

        TagAddDescriptor descriptor = new TagAddDescriptor(person);
        descriptor.setTags(tagSet);
        TagAddCommand command = (TagAddCommand) parser.parseCommand(TagAddCommand.COMMAND_WORD + " "
            + INDEX_FIRST_PERSON.getOneBased() + " friend");
        assertEquals(new TagAddCommand(singlePersonIndexList, descriptor), command);
    }

    @Test
    public void parseCommandTagFind() throws Exception {
        TagMatchingKeywordPredicate predicate = new TagMatchingKeywordPredicate("friend");
        TagFindCommand command = (TagFindCommand) parser.parseCommand(TagFindCommand.COMMAND_WORD
                + " friend");
        assertEquals(new TagFindCommand(predicate), command);
    }

    @Test
    public void parseCommandTagRemove() throws Exception {
        Person person = new PersonBuilder().build();
        ArrayList<Index> singlePersonIndexList = new ArrayList<>();
        singlePersonIndexList.add(INDEX_FIRST_PERSON);
        Set<Tag> tagSet = new HashSet<>();
        tagSet.add(new Tag("friend"));

        TagRemoveDescriptor descriptor = new TagRemoveDescriptor(person);
        descriptor.setTags(tagSet);
        TagRemoveCommand command = (TagRemoveCommand) parser.parseCommand(TagRemoveCommand.COMMAND_WORD + " "
                + INDEX_FIRST_PERSON.getOneBased() + " friend");
        assertEquals(new TagRemoveCommand(singlePersonIndexList, descriptor), command);
    }

    @Test
    public void parseCommand_exit() throws Exception {
        assertTrue(parser.parseCommand(ExitCommand.COMMAND_WORD) instanceof ExitCommand);
        assertTrue(parser.parseCommand(ExitCommand.COMMAND_WORD + " 3") instanceof ExitCommand);
    }

    @Test
    public void parseCommand_find() throws Exception {
        List<String> keywords = Arrays.asList("n/", "foo", "bar", "baz");
        FindCommand command = (FindCommand) parser.parseCommand(
                FindCommand.COMMAND_WORD + " " + keywords.stream().collect(Collectors.joining(" ")));
        assertEquals(new FindCommand(new NameContainsKeywordsPredicate(keywords)), command);
    }

    @Test
    public void parseCommand_search() throws Exception {
        List<String> keywords = Arrays.asList("n/", "foo", "bar", "baz");
        FindCommand command = (FindCommand) parser.parseCommand(
                FindCommand.COMMAND_WORD_2 + " " + keywords.stream().collect(Collectors.joining(" ")));
        assertEquals(new FindCommand(new NameContainsKeywordsPredicate(keywords)), command);
    }

    @Test
    public void parseCommand_get() throws Exception {
        List<String> keywords = Arrays.asList("n/", "foo", "bar", "baz");
        FindCommand command = (FindCommand) parser.parseCommand(
                FindCommand.COMMAND_WORD_3 + " " + keywords.stream().collect(Collectors.joining(" ")));
        assertEquals(new FindCommand(new NameContainsKeywordsPredicate(keywords)), command);
    }

    @Test
    public void parseCommand_help() throws Exception {
        assertTrue(parser.parseCommand(HelpCommand.COMMAND_WORD) instanceof HelpCommand);
        assertTrue(parser.parseCommand(HelpCommand.COMMAND_WORD + " 3") instanceof HelpCommand);
    }

    @Test
    public void parseCommand_second() throws Exception {
        assertTrue(parser.parseCommand(HelpCommand.COMMAND_WORD_2) instanceof HelpCommand);
        assertTrue(parser.parseCommand(HelpCommand.COMMAND_WORD_2 + " 3") instanceof HelpCommand);
    }

    @Test
    public void parseCommand_history() throws Exception {
        assertTrue(parser.parseCommand(HistoryCommand.COMMAND_WORD) instanceof HistoryCommand);
        assertTrue(parser.parseCommand(HistoryCommand.COMMAND_WORD + " 3") instanceof HistoryCommand);

        try {
            parser.parseCommand("histories");
            fail("The expected ParseException was not thrown.");
        } catch (ParseException pe) {
            assertEquals(MESSAGE_UNKNOWN_COMMAND, pe.getMessage());
        }
    }

    @Test
    public void parseCommand_record() throws Exception {
        assertTrue(parser.parseCommand(HistoryCommand.COMMAND_WORD_2) instanceof HistoryCommand);
        assertTrue(parser.parseCommand(HistoryCommand.COMMAND_WORD_2 + " 3") instanceof HistoryCommand);

        try {
            parser.parseCommand("histories");
            fail("The expected ParseException was not thrown.");
        } catch (ParseException pe) {
            assertEquals(MESSAGE_UNKNOWN_COMMAND, pe.getMessage());
        }
    }

    @Test
    public void parseCommand_list() throws Exception {
        assertTrue(parser.parseCommand(ListCommand.COMMAND_WORD) instanceof ListCommand);
        assertTrue(parser.parseCommand(ListCommand.COMMAND_WORD + " 3") instanceof ListCommand);
    }

    @Test
    public void parseCommand_show() throws Exception {
        assertTrue(parser.parseCommand(ListCommand.COMMAND_WORD_2) instanceof ListCommand);
        assertTrue(parser.parseCommand(ListCommand.COMMAND_WORD_2 + " 3") instanceof ListCommand);
    }

    @Test
    public void parseCommand_all() throws Exception {
        assertTrue(parser.parseCommand(ListCommand.COMMAND_WORD_3) instanceof ListCommand);
        assertTrue(parser.parseCommand(ListCommand.COMMAND_WORD_3 + " 3") instanceof ListCommand);
    }

    @Test
    public void parseCommand_select() throws Exception {
        SelectCommand command = (SelectCommand) parser.parseCommand(
                SelectCommand.COMMAND_WORD + " " + INDEX_FIRST_PERSON.getOneBased());
        assertEquals(new SelectCommand(INDEX_FIRST_PERSON), command);
    }

    @Test
    public void parseCommand_choose() throws Exception {
        SelectCommand command = (SelectCommand) parser.parseCommand(
                SelectCommand.COMMAND_WORD_2 + " " + INDEX_FIRST_PERSON.getOneBased());
        assertEquals(new SelectCommand(INDEX_FIRST_PERSON), command);
    }

    @Test
    public void parseCommand_map_show() throws Exception {
        MapShowCommand command = (MapShowCommand) parser.parseCommand(
                MapShowCommand.COMMAND_WORD + " " + INDEX_FIRST_PERSON.getOneBased());
        assertEquals(new MapShowCommand(INDEX_FIRST_PERSON), command);
    }

    @Test
    public void parseCommand_map_route() throws Exception {
        String startLocation = "Clementi Street";
        MapRouteCommand command = (MapRouteCommand) parser.parseCommand(
                MapRouteCommand.COMMAND_WORD + " " + INDEX_FIRST_PERSON.getOneBased() + " "
                        + PREFIX_ADDRESS + startLocation);
        assertEquals(new MapRouteCommand(INDEX_FIRST_PERSON, startLocation), command);
    }

    @Test
    public void parseCommand_redoCommandWord_returnsRedoCommand() throws Exception {
        assertTrue(parser.parseCommand(RedoCommand.COMMAND_WORD) instanceof RedoCommand);
        assertTrue(parser.parseCommand("redo 1") instanceof RedoCommand);
    }

    @Test
    public void parseCommand_undoCommandWord_returnsUndoCommand() throws Exception {
        assertTrue(parser.parseCommand(UndoCommand.COMMAND_WORD) instanceof UndoCommand);
        assertTrue(parser.parseCommand("undo 3") instanceof UndoCommand);
    }

    @Test
    public void parseCommand_unrecognisedInput_throwsParseException() throws Exception {
        thrown.expect(ParseException.class);
        thrown.expectMessage(String.format(MESSAGE_INVALID_COMMAND_FORMAT, HelpCommand.MESSAGE_USAGE));
        parser.parseCommand("");
    }

    @Test
    public void parseCommand_unknownCommand_throwsParseException() throws Exception {
        thrown.expect(ParseException.class);
        thrown.expectMessage(MESSAGE_UNKNOWN_COMMAND);
        parser.parseCommand("unknownCommand");
    }
}
```
###### /java/seedu/address/logic/parser/DeleteCommandParserTest.java
``` java

/**
 * As we are only doing white-box testing, our test cases do not cover path variations
 * outside of the DeleteCommand code. For example, inputs "1" and "1 abc" take the
 * same path through the DeleteCommand, and therefore we test only one of them.
 * The path variation for those two cases occur inside the ParserUtil, and
 * therefore should be covered by the ParserUtilTest.
 */
public class DeleteCommandParserTest {

    private DeleteCommandParser parser = new DeleteCommandParser();

    @Test
    public void parse_validArgs_returnsDeleteCommand() {
        ArrayList<Index> todelete = new ArrayList<>();
        todelete.add(INDEX_FIRST_PERSON);
        assertParseSuccess(parser, "I/1", new DeleteCommand(todelete));
    }

    @Test
    public void parse_validArgs_returnsDeleteCommand1() {
        ArrayList<Index> todelete = new ArrayList<>();
        todelete.add(INDEX_FIRST_PERSON);
        todelete.add(INDEX_SECOND_PERSON);
        assertParseSuccess(parser, "I/1 2", new DeleteCommand(todelete));
    }

    @Test
    public void parse_validArgs_returnsDeleteCommand2() {
        ArrayList<Index> todelete = new ArrayList<>();
        todelete.add(INDEX_FIRST_PERSON);
        assertParseSuccess(parser, "n/Alice Pauline", new DeleteCommand("Alice Pauline"));
    }

    @Test
    public void parse_invalidArgs_throwsParseException() {
        assertParseFailure(parser, "I/", String.format(MESSAGE_INVALID_COMMAND_FORMAT, DeleteCommand.MESSAGE_USAGE));
    }

    @Test
    public void parse_emptyArgs_throwsParseException() {
        assertParseFailure(parser, "", String.format(MESSAGE_INVALID_COMMAND_FORMAT, DeleteCommand.MESSAGE_USAGE));
    }

    @Test
    public void parse_invalidstartArgs_throwsParseException() {
        assertParseFailure(parser, "aI/", String.format(MESSAGE_INVALID_COMMAND_FORMAT, DeleteCommand.MESSAGE_USAGE));
    }

    @Test
    public void parse_invalidArgs_throwsParseException1() {
        assertParseFailure(parser, "n/", String.format(MESSAGE_INVALID_COMMAND_FORMAT, DeleteCommand.MESSAGE_USAGE));
    }


    @Test
    public void parse_invalidArgs_throwsParseException2() {
        assertParseFailure(parser, "a", String.format(MESSAGE_INVALID_COMMAND_FORMAT, DeleteCommand.MESSAGE_USAGE));
    }
}
```
###### /java/seedu/address/logic/parser/HelpCommandParserTest.java
``` java

public class HelpCommandParserTest {
    private HelpCommandParser parser = new HelpCommandParser();

    @Test
    public void parsesuccess() {
        assertParseSuccess(parser, AddCommand.COMMAND_WORD, new HelpCommand("add"));

        assertParseSuccess(parser, AddCommand.COMMAND_WORD_2, new HelpCommand("add"));

        assertParseSuccess(parser, AddCommand.COMMAND_WORD_3, new HelpCommand("add"));

        assertParseSuccess(parser, ClearCommand.COMMAND_WORD, new HelpCommand("clear"));

        assertParseSuccess(parser, DeleteCommand.COMMAND_WORD, new HelpCommand("delete"));

        assertParseSuccess(parser, DeleteCommand.COMMAND_WORD_2, new HelpCommand("delete"));

        assertParseSuccess(parser, DeleteCommand.COMMAND_WORD_3, new HelpCommand("delete"));

        assertParseSuccess(parser, EditCommand.COMMAND_WORD, new HelpCommand("edit"));

        assertParseSuccess(parser, EditCommand.COMMAND_WORD_2, new HelpCommand("edit"));

        assertParseSuccess(parser, EditCommand.COMMAND_WORD_3, new HelpCommand("edit"));

        assertParseSuccess(parser, ExitCommand.COMMAND_WORD, new HelpCommand("exit"));

        assertParseSuccess(parser, FindCommand.COMMAND_WORD, new HelpCommand("find"));

        assertParseSuccess(parser, FindCommand.COMMAND_WORD_2, new HelpCommand("find"));

        assertParseSuccess(parser, FindCommand.COMMAND_WORD_3, new HelpCommand("find"));

        assertParseSuccess(parser, HistoryCommand.COMMAND_WORD, new HelpCommand("history"));

        assertParseSuccess(parser, HistoryCommand.COMMAND_WORD_2, new HelpCommand("history"));

        assertParseSuccess(parser, ListCommand.COMMAND_WORD, new HelpCommand("list"));

        assertParseSuccess(parser, ListCommand.COMMAND_WORD_2, new HelpCommand("list"));

        assertParseSuccess(parser, ListCommand.COMMAND_WORD_3, new HelpCommand("list"));

        assertParseSuccess(parser, RedoCommand.COMMAND_WORD, new HelpCommand("redo"));

        assertParseSuccess(parser, SelectCommand.COMMAND_WORD, new HelpCommand("select"));

        assertParseSuccess(parser, SelectCommand.COMMAND_WORD_2, new HelpCommand("select"));

        assertParseSuccess(parser, SortCommand.COMMAND_WORD, new HelpCommand("sort"));

        assertParseSuccess(parser, TagAddCommand.COMMAND_WORD, new HelpCommand("tagadd"));

        assertParseSuccess(parser, TagRemoveCommand.COMMAND_WORD, new HelpCommand("tagremove"));

        assertParseSuccess(parser, UndoCommand.COMMAND_WORD, new HelpCommand("undo"));

    }

}
```