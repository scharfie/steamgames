require 'nokogiri'
require 'open-uri'

class URLCache
  def self.fetch(key, url)
    filename = "cache/#{key}"
    age = Time.now - 3600

    if File.exists?(filename) && File.mtime(filename) > age
      data = File.read(filename)
    else
      data = open(url)
      File.write(filename, data)
    end

    return data
  end
end

class Steam
  class Game
    attr_accessor :name

    def initialize(h={})
      h.each { |k, v| send(:"#{k}=", v) }
    end
  end

  def self.games(steamid)
    data = URLCache.fetch("games-#{steamid}.xml", "https://steamcommunity.com/id/#{steamid}/games?tab=all&xml=1")
    xml = Nokogiri.XML(data)

    games = xml.css("game > name").map do |node|
      Game.new(
        name: node.inner_text
      )
    end

    return games
  end
end

task :default => :tsv

desc "Generates TSV list of games and copies result to clipboard"
task :tsv do
  steamid = ENV.fetch("STEAM_ID")
  result = ""
  games = Steam.games(steamid)
  result << "Name\tLibrary\t(updated: #{Date.today})"
  games.each do |game|
    result << "\n%s\t%s" % [game.name, "Steam"]
  end

  IO.popen('pbcopy', 'w') { |f| f << result }
  puts result
end
