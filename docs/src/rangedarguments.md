# Ranged arguments

Ranged arguments allow players to provide a range between two numbers, all within a single argument. The CommandAPI provides two ranged arguments, `IntegerRangeArgument` for ranges with only integer values, and `FloatRangeArgument` for ranged with potential floating point values.

These consist of values such as:

| Input   | What it means                                                |
| ------- | ------------------------------------------------------------ |
| `5`     | The number 5                                                 |
| `5..10` | Numbers between 5 and 10, including 5 and 10                 |
| `5..`   | Numbers greater than or equal to 5 (bounded by Java's max number size) |
| `..5`   | Numbers less than or equal to 5 (bounded by Java's min number size) |

## The IntegerRange & FloatRange class

The CommandAPI returns an `IntegerRange` from the `IntegerRangeArgument`, and a `FloatRange` from the `FloatRangeArgument`, which represents the upper and lower bounds of the numbers provided by the command sender, as well as a method to check if a number is within that range.

The `IntegerRange` class has the following methods:

```java
class IntegerRange {
    public int getLowerBound();
    public int getUpperBound();
    public boolean isInRange(int);
}
```

The `FloatRange` class has the following methods:

```java
class FloatRange {
    public float getLowerBound();
    public float getUpperBound();
    public boolean isInRange(float);
}
```

## Example - Searching chests for certain items

Say you're working on a plugin for server administrators to help them find illegal items. A method of doing so would be to search chests for certain items.

```
/searchchests <range> <item>
```

```java
// Declare our arguments for /searchrange <IntegerRange> <ItemStack>
LinkedHashMap<String, Argument> arguments = new LinkedHashMap<>();
arguments.put("range", new IntegerRangeArgument());
arguments.put("item", new ItemStackArgument());

CommandAPI.getInstance().register("searchrange", arguments, (sender, args) -> {
	// Retrieve the range from the arguments
	IntegerRange range = (IntegerRange) args[0];
	ItemStack itemStack = (ItemStack) args[1];
	
	// Inform the sender that they must be a player
	if(!(sender instanceof Player)) {
		sender.sendMessage("You must be a player to run this command!");
		return;
	}
	
	Player player = (Player) sender;
	
	// Store the locations of chests with certain items
	List<Location> locations = new ArrayList<>();
	
	// Iterate through all chunks, and then all tile entities within each chunk
	for(Chunk chunk : player.getWorld().getLoadedChunks()) {
		for(BlockState blockState : chunk.getTileEntities()) {
			
			// The distance between the block and the player
			int distance = (int) blockState.getLocation().distance(player.getLocation());
			
			// Check if the distance is within the specified range 
			if(range.isInRange(distance)) {
				
				// Check if the tile entity is a chest
				if(blockState instanceof Chest) {
					Chest chest = (Chest) blockState;
					
					// Check if the chest contains the item specified by the player
					if(chest.getInventory().contains(itemStack.getType())) {
						locations.add(chest.getLocation());
					}
				}
			}
			
		}
	}
	
	// Output the locations of the chests, or whether no chests were found
	if(locations.isEmpty()) {
		player.sendMessage("No chests were found");
	} else {
		player.sendMessage("Found " + locations.size() + " chests:");
		locations.forEach(location -> {
			player.sendMessage("  Found at: " 
					+ location.getX() + ", " 
					+ location.getY() + ", " 
					+ location.getZ());
		});
	}
});
```