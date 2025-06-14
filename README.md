# blazor-validation

model name and property name come as strings from an API response

{
  "errors": {
    "Name": ["Name is required"],
    "Address.City": ["City is required"]
  }
}


public class Person
{
    public string Name { get; set; }
    public AddressInfo Address { get; set; } = new();
}

public class AddressInfo
{
    public string City { get; set; }
}

<EditForm EditContext="@editContext" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <InputText @bind-Value="person.Name" />
    <ValidationMessage For="@(() => person.Name)" />

    <InputText @bind-Value="person.Address.City" />
    <ValidationMessage For="@(() => person.Address.City)" />

    <button type="submit">Submit</button>
</EditForm>

@code {
    private Person person = new();
    private EditContext editContext;
    private ValidationMessageStore messageStore;

    protected override void OnInitialized()
    {
        editContext = new EditContext(person);
        messageStore = new ValidationMessageStore(editContext);
    }

    private void HandleValidSubmit()
    {
        // Simulate server-side validation response
        var apiErrors = new Dictionary<string, List<string>>
        {
            { "Name", new List<string> { "Name is required" } },
            { "Address.City", new List<string> { "City is required" } }
        };

        ApplyApiErrors(apiErrors);
    }

    private void ApplyApiErrors(Dictionary<string, List<string>> errors)
    {
        messageStore.Clear();

        foreach (var error in errors)
        {
            var (model, propertyName) = ResolveModelAndProperty(person, error.Key);
            if (model != null)
            {
                var field = new FieldIdentifier(model, propertyName);
                messageStore.Add(field, error.Value);
            }
        }

        editContext.NotifyValidationStateChanged();
    }

    private (object model, string propertyName) ResolveModelAndProperty(object rootModel, string path)
    {
        var parts = path.Split('.');
        object current = rootModel;

        for (int i = 0; i < parts.Length - 1; i++)
        {
            var prop = current?.GetType().GetProperty(parts[i]);
            if (prop == null) return (null, null);
            current = prop.GetValue(current);
        }

        return (current, parts[^1]); // return model object and final property name
    }
}

