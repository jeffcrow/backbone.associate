Backbone Relations 
==================

Given a bunch of related models and collections:

    var Country = Backbone.Model.Extend({ /* ... */ });
    var Flag = Backbone.Model.Extend({ /* ... */ });
    var City = Backbone.Model.Extend({ /* ... */ });
    var Cities = Backbone.Collection.extend({ model: City });

It would be nice to have a simple way to describe the relationships 
between them. We don't need anything fancy.

    Backbone.associate(Country, {
      flag: { type: Flag },
      cities: { type: Cities }
    });

Now, whenever data is parsed, child resources can be instantiated 
and stored in the attribute hash:

    var data = {
      cities: [
        { name: 'Calgary' },
        { name: 'Regina' }
      ]
    }

    var canada = new Country(data, { parse: true });

When it's time to sync the parent resource back up with the
database, child resources will be serialized (by key) and
included in the request.

    canada.toJSON(); // { flag: { colors: ['red','white'] }, ...

Since associates are *just attributes*, they may be accessed at any 
time using the usual `get` method:

    var cities = canada.get('cities');

Or through the sticky-sweet goodness of a sugary accessor:

    canada.flag().set({ colors: ['red','white'] });
    canada.cities().add([
      { name: 'Edmonton' },
      { name: 'Montreal' },
      { name: 'Ottawa' },
      { name: 'Vancouver' }
    ]);

That's handy for manipulating the relations, setting up eventing, or 
any of the many other things this plugin won't do for you.

### Things this plugin won't do for you...

..include managing children during `set` operations, configuring child 
URLs, and making presumptions about child events. Fortunately, all of 
these can be implemented as needed:

    // manage `set` operations
    Country.prototype.set = function (attributes) {

      // for a collection
      if (_.has(attributes, 'cities')) {
        this.cities().reset(attributes.cities);
        delete attributes.cities;
      }

      // for a model
      if (_.has(attributes, 'flag')) {
        this.flag().set(attributes.flag);
        delete attributes.flag;
      }

      return Backbone.Model.prototype.set.call(this, attributes);
    };

    // configure child URLs
    canada.cities().urlRoot = canada.url() + '/cities'
    canada.flag().url = canada.url() + '/flag'

    // handle child events
    canada.listenTo(canada.cities(), 'change', canada.onCityChanged);

## Contributing

Contributions are welcome!

1. Fork this repo
2. Add your changes and update the spec as needed
3. Submit a [pull request](help.github.com/pull-requests/)
